# RFC 137: support subsuites in wptrunner

## Summary

Support running tests in multiple different browser configurations in
a single wptrunner invocation.

## Details

It is common for browser vendors to want to run web platform tests
with particular configurations of the browser. For example when
developing a major new feature, or refactoring existing code, they may
want to run with non-default configuration to enable the new
code paths. Often, for cost and efficiency reasons, these test runs
will initially be limited to just the tests most likely to be affected
by the changes e.g. a change to session history is highly likely to
affect tests involving navigation, but highly unlikely to affect CSS
layout.

Typically vendors wanting to enable multiple configurations had to
define those configurations externally, and invoke wptrunner multiple
times, once per configuration. This has some advantages; in particular
it keeps the vendor-specific issue of browser configurations cleanly
separated from the actual business of running tests. However there are
also a couple of disadvantages:

* When configurations are handled externally, they are either written
  into a CI configuration, which is then difficult to run locally, or
  the vendor is responsible for writing a substantial frontend on top
  of wptrunner which handles invoking it for each of the different
  configurations.

* Even with a vendor frontend it's difficult to schedule the tests
  optimally across multiple cores. This is because you essentially
  have two choices on how to handle the different
  configurations. Option one is to run each configuration in serial,
  with each wptrunner process using all available cores. This can be
  wasteful when you have many configurations, some of which only run a
  small number of tests, because those configurations only use a small
  fraction of the available compute, but can't be scheduled in
  parallel. Option two is to invoke many wptrunner in parallel processes, each
  responsible for running a small fraction of the total tests in each
  configuration. However wptrunner does a lot of global setup (reading
  the test manifest, starting wptserve), and repeating this work per
  process would be very inefficient.

So in order to meet this use case there are two obvious options:

1. Refactor wptrunner to allow it to be driven by an external test
   scheduler without having to redo all the global setup for each
   group of tests.

1. Push the notion of multiple configurations down into wptrunner
   itself, and update the existing test scheduling logic to handle
   multiple browser configurations.

In practice people are currently using wptrunner as a monolith, with
test selection and browser setup provided by command line options and
configuration files, so although the architecture would probably allow
adopting a more service-based model, it's more in keeping with existing
usage patterns to implement support for multiple configurations
directly in wptrunner.

### Implementation Strategy

From an implementation point of view, the main constraint is around
how we associate test results with a specific configuration.

Our test data flow is built around mozlog. This groups tests into
"suites". Each suite is started with a `suite_start` event ends with a
`suite_end` event. The `suite_start` event contains information,
particularly in the `run_info` key about the browser being tested. That
`run_info` data is used in wptrunner metadata files to determine which
metadata applies to a specific test run (e.g. to allow different
expected results on different platforms). Specific test results are
associated to a suite implicitly by sequence i.e. any `test_start` log
event coming after a specific `suite_start` is grouped within that
suite. This implicitly means that only a single suite can be running
in the context of a specific logger at a specific time. That is
incompatible with the requirement to run multiple configurations in
parallel.

There are three obvious approaches we could use to address this:

1. Use multiple loggers, one per configuration. In this case each
   configuration would produce a separate stream of log events that
   could be written to a separate file. However mozlog is really
   designed around the assumption of a single top-level logger per
   process, which is reflected in the way output is configured with
   command line options. Since we'd presumably still want a single
   top-level output to stdout, we'd still need to solve the problem of
   combining multiple log streams into one top-level stream, which
   would require a mechanism to relax the one-suite-at-a-time
   constraint. We would also need to work out how to pass specific
   configuration (e.g. output filenames) down to specific loggers.

1. Put configuration information inside the test id, so that from the
   point of view of consumers, the same test run in different
   configurations look like different tests. This is how the Chromium
   CI system handles this problem with the concept of "virtual"
   testsuites. It has the advantage that it requires minimal changes
   to data structures; tests get an id like `/<prefix>/<configuration
   name>/<actual test id>`, so from the point of view of consumers
   these are different tests. However this is also a disadvantage;
   every consumer that wants to process the results (e.g. to compare
   results between configurations) has to know about the details of
   how to parse the test id to extract the "real" test and
   configuration name. This goes directly against the mozlog design
   principle of providing data in a structured form to minimise the
   amount of error-prone parsing required in consumers. It also makes
   these configurations work in a way that's markedly different to
   other kinds of configuration data (e.g. platform name, whether
   we're running a debug build, etc.); that data is represented in the
   `run_info` rather than as part of the test id.

1. Add the concept of multiple "subsuites" directly in the mozlog data
   model and explicitly link other log actions which depend on the
   configuration to the subsuite. This has the disadvantage that it
   requires updates to consumers which care about subsuite membership,
   but the advantage that over the longer term there's a clearly
   defined data model which doesn't require additional parsing to
   understand test identity. It also allows us to treat subsuite
   configuration like other kinds of configuration data, by
   associating each subsuite with particular `run_info` data.

The proposal is to adopt the third approach, and update mozlog to
allow it to represent the concept of subsuites, and wptrunner to allow
scheduling tests from multiple subsuites in parallel.

### Mozlog Changes

For mozlog, we propose allowing each top level test suite (represented
by a `suite_start` / `suite_end` event pair) to also have associated
subsuites. Each subsuite will define updates to the top-level suite
`run_info` data for the configuration in that subsuite. Log actions
which are associated with a specific test or set of tests will get a
`subsuite` property that explicitly names the subsuite they're
associated with.

In detail this means one extra log event:

```py
add_subsuite(name: str, run_info: Mapping[str, Any])
```

`name` is required and must be unique within a given suite. `run_info`
is optional and represents an update to the `run_info` of the
suite. In addition the logger adds a `subsuite` key to the `run_info`,
set to the name of the `subsuite` (TODO: should this be implicit, or
should consumers get freedom in how to handle this?).

Plus `subsuite` keys added to the following log events:

* `test_start`
* `test_end`
* `test_status`
* `assertion_count`
* `lsan_leak`
* `lsan_summary`
* `mozleak_object`
* `mozleak_total`

In all cases this key is optional, and when missing it's assumed that
the event is associated with the default (unnamed) subsuite. Mozlog
will enforce the property that the subsuite name corresponds to a
subsuite that has previously been added with `add_subsuite`.

### wptrunner changes

wptrunner already has the concept of test groups. These are sets of
tests which run in a single browser process, between which the browser
is restarted. Parallel test running is enabled by scheduling multiple
test groups, each of which is associated with a single browser
instance.

It also has the concept of multiple test types (e.g. testharness,
reftest), each of which has a different implementation. Each test
group is associated with a single type of test (i.e. a single group
can contain either testharness tests or reftests, but not both)
i.e. we can form a mapping from a test type to a list of test
groups. Groups of different test types can be run in parallel.

For subsuites the different browser configurations in each subsuite
mean that it's generally desirable not to reuse browser instances
between different subsuites either by necessity (e.g. because the
configuration of the browser for the subsuite has to happen at browser
startup) or because of isolation (so that we don't accidentally get
shared browser state between subsuites affecting the results of
later-run tests). Therefore we can assume that each test group should
be associated with exactly one subsuite. This leaves a small amount of
parallelism on the table in cases where it would be possible to run
multiple subsuites in the same group, but the resulting model is
relatively easy to understand and satisfies the main use cases where
changing configuration at runtime is not possible.

Therefore the implementation will work as follows:

* Test groups are associated with a pair `(subsuite, test_type)`
  rather than a test type alone.

* Each `(subsuite, test_type)` combination has a unique
  `TestImplementation`, which corresponds to a unique instance of the
  product-specific `Browser` and `Executor` classes.

* Each product implementation is responsible for configuring the
  browser and executor classes as required for the subsuite. To
  facilitate this the subsuite details are passed to the product
  specific functions for getting the browser and executor kwargs.

* Any global state (e.g. reftest screenshot cache) that depends on the
  test id is instead keyed on `(subsuite, test.id)`.

#### Subsuite definition

The definition of subsuites contains the following properties:

* `name` (required) - The name of the subsuite.

* `config` (required) - Product-specific configuration options for the
  subsuite. This is used to e.g. set the command line arguments used
  when launching the browser, or the prefs set in the browser profile.

* `run_info` (optional) - Any updates to the `run_info` associated with the
  subsuite, other than its name (the `run_info["subsuite"] = name` key
  is set implicitly).

* `include` (optional) - A list of test id prefixes to include when
  running the subsuite.

* `tags` (optional) - A list of test tags (as defined in the metadata
  files) to include when running the subsuite.

* `expires` (optional) - A date after which the subsuite is no longer
  run (required for compatibility with Chromium's existing
  infrastructure).

In the first instance the subsuite definitions will be provided in a
JSON file of the form

```json
{
  subsuite_name: {properties}
}
```

This will be passed to wptrunner using a new `--subsuite-file` command
line argument.

#### Test selection

The test selection mechanism in wptrunner is complex, and tries to
meet many use cases. Adding subsuites into the mix inevitably
increases that complexity.

Typically in wptrunner test loading happens in multiple stages:

1. One or more `MANIFEST.json` files are loaded, providing a complete list of
   all the tests in the suite (multiple `MANIFEST.json` files are used
   to allow vendors to add vendor-specific tests to the suite).

1. The entries of each manifest are iterated. These are filtered
   according to a test id filter built from the `--include`,
   `--exclude`, `--include-file` and `--include-manifest` command line
   arguments.

1. For each included test, the corresponding metadata is parsed.

1. Further filters based on the metadata values are applied. The
   `--tag` command line argument, and `--skip-implementation-status`
   command line arguments are appleid at this stage, since they depend
   on metadata values.

1. The tests are split into test groups, according to the
   configuration.

The `--test-groups` command line argument changes the behaviour here
slightly; it directly provides a list of tests that will run, and
names groups for the tests in the format `{group_name:
[tests_ids]}`. This can be used in combination with the `--include`
command line argument to subset the tests in the file, but trying to
specify more tests than in the file will cause an error.

The current metadata processing means that we need to read the
metadata files once per subsuite (since we resolve the metadata
properties when reading the file, and the metadata can be different
for different subsuites).

The simplest solution here is to read the `MANIFEST.json` files once,
apply the top-level filters that apply to all subsuites (e.g. those
from `--include`), and then run the remaining steps for each subsuite,
adding the subsuite include filters after parsing the metadata. This
isn't the optimal solution, since we can end up reading metadata
multiple times even though it's not used. However further
optimisations are possible (e.g. separating out the parse and
evaluation phase of reading metadata, as we do with the update tool,
making the top-level filters per-subsuite to avoid reading metadata
files for tests won't be included in the subsuite). However these
optimistations should be possible without backwards-incompatible
changes.

Since the `--test-groups` file is directly providing a list of tests
that run, rather than a filter, the format of that file is modified in
a backwards compatible way so that instead of the keys being just
group name, they are of the form `<subsuite_name>:<group_name>`, with
the part up to and including the colon omitted for the default subsuite.

Running individual subsuites will be possible with the `--subsuite`
command line argument. This can be provided multiple times to run
multiple subsuites.

#### wptreport.json

The wptreport.json format needs to be updated to allow specifying
which subsuite each test belongs to. To do this two principal changes
are proposed:

1. A new top-level key `subsuites` which contains a mapping from
   subsuite name to the `run_info` data specific to that subsuite.

1. Entries in the `results`, `asserts`, `lsan_leaks`, and `mozleak`
   top-level fields get a `subsuite` property which is the name of a
   subsuite.

#### Metadata update

The automatic metadata update works by collecting test results from
multiple files, grouped by their `run_info` data. This can naturally
be extended to support subsuites, just by extending the processing to
create a `run_info` entry per subsuite rather than assuming the
`run_info` is the same for all tests in a file.

The main difficulty here is that the metadata expression language
doesn't have a concept of `null`. So we need to represent the default
subsuite name as the empty string in `run_info`, or add a `null`
concept to allow specifying metadata specific to the default subsuite.

## Risks

The main trade-offs of the proposed approach have been discussed in the
Details section.

The main risk is that by making changes to the underlying data model
we need to update tooling that consumes the generated files. However,
because the changes are purely additive, and the use-cases for
subsuites are limited (e.g. we don't currently have any plans to use
these on the GitHub infrastructure), it should be possible to upgrade
consumers as required. In particular no changes proposed here should
affect wpt.fyi in the foreseeable future.
