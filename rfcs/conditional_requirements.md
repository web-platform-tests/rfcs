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
a subcommand may need a third party library only when a specific command line
flag is specified.

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
  ...
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
`--enable-webtransport-h3` command line flag is provided.

### Follow Ups

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
