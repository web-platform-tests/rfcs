# RFC 119: Allow wptrunner to retry tests with unexpected results

## Summary

Add a `--retry-unexpected=<n>` option to wptrunner to retry each test with
unexpected initial results up to `n` times or until it runs as expected.
`n` defaults to zero.

## Details

### Rationale

Retries are intended for CI systems that should be robust to false positives
that unnecessarily block developers:
* Transient infrastructure issues (e.g., CPU utilization spike causing
  timeouts).
* Incomplete expectation data checked into the trunk of a browser vendor's
  source control system.
  The CI check might not surface statuses that should have been added.

### Proposal

To gather initial results, wptrunner will run tests according to `--repeat`
and `--rerun` as it does currently.
When passed `--retry-unexpected`, wptrunner will retry tests that gave an
unexpected test or subtest status in every repeat iteration.
In a retry, a test takes any expected status as its final result and is not
retried in subsequent iterations.
The retry loop can halt early if all tests have been exonerated.
wptrunner can return a successful exit code if retries were enabled and all
tests eventually ran as expected.

Because the purpose of retries is to separate flakiness from consistently
unexpected results, there is no need to retry tests that ran as expected in at
least one repeat iteration.

A retry iteration works as a repeat iteration does (e.g., emits
`suite_{start,end}` log events), except the `rerun` option is forced to 1.
Requiring restarts eliminates bad state carried over as a possible cause of
failure.

### Alternatives

* Using `--repeat` alone and only failing the CI check when a test runs
  unexpectedly consistently.
  Given that almost all tests should run as expected the first time, this scheme
  unnecessarily burdens the CI infrastructure.
* Implement retries in vendor CI systems.
  Browser vendors will not benefit from a shared implementation native to
  wptrunner.
  This also adds complexity downstream for merging test results from multiple
  runs.
* A `--no-repeat-expected` that modifies `--repeat` to skip tests that run
  expectedly.
  Retries have different semantics, including changing when wptrunner exits with
  a successful status code.

## Risks

Minimal, since this is an opt-in feature.
Existing behavior will not change.

## Appendix

[Prototype](https://github.com/web-platform-tests/wpt/compare/master...jonathan-j-lee:wpt:no-repeat-expected)
