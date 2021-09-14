# Conditional requirements support in wptrunner

## Summary

Add a new field `conditional_requirements` in commands.json so that conditional
requirements can be specified for `wpt` subcommands.

## Details

### Context

`wpt` uses commands.json to set up subcommands (see `load_commands()` in
tools/wpt/wpt.py). A subcommand can specify
[requirements.txt](https://pip.pypa.io/en/stable/user_guide/#requirements-files)
to install its dependencies in a virtualenv environment. These dependencies are
unconditionally installed before running the subcommand.

There is a demand to install additional dependencies conditionally. For example,
supporting conditional requirements can be helpful in the following situation:

* A subcommand supports a command line flag to enable a specific feature.
* The feature needs third party libraries.
* These libraries also have dependencies.
* The feature is experimental and some browser testing infrastructures may not
  want to introduce such dependencies yet.

A specific example of the above situation is
[WebTransport over HTTP/3 support](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/webtransport_h3_test_server.md).
The WebTransport over HTTP/3 server requires
[aioquic](https://aioquic.readthedocs.io/en/latest/) and it also has several
dependencies.

```sh
# This doesn’t require `aioquic` library
$ ./wpt run
# This requires `aioquic` library
$ ./wpt run --enable-webtransport-h3
```

This RFC proposes the `conditional_requirements` field in commands.json to
support conditional requirements like the above example.

### commands.json addition

A value of `conditional_requirements` would look like the following:

```json
{
  "conditional_requirements": {
    "commandline_flag": {
      "enable_webtransport_h3": [
        "../webtransport/requirements.txt"
      ]
    }
  }
}
```

A `conditional_requirements` contains key value pairs. This RFC only defines
`commandline_flag` as a key. `commandline_flag` is examined when the
corresponding command line flag is provided. In the above example, `wpt`
installs requirements in `../webtransport/requirements.txt` only when the
`--enable-webtransport-h3` command line flag is provided for the subcommand.

### Alternatives considerered

Conditional requirements support introduces complexity to some extent.
Installing all potential dependencies unconditionally could be an alternative
approach. The upside of the alternative approach is that we keep `wptrunner` as
simple as possible. The downside is that installing unnecessary dependencies can
also be a source of the maintenance burden.

Conditional requirements support gives us a way to take an incremental approach
to introduce dependencies suitable for each testing infrastructure.

### Follow ups

This proposal could in theory generalize it to allow supporting the wptrunner
use case for installing per-product requirements. It might look like:

```json
{
  "conditional_requirements": {
    "product": {
      "chrome": [
        "requirements_chrome.txt"
      ]
    }
  }
}
```

## Risk

Adding more fields in `commands.json` makes `wpt` complicated to execute
subcommands and may lead to increasing maintenance costs. We improve
documentation and describe how `commands.json` is used.
