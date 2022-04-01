# RFC 110: support virtual test suites in wptrunner

## Summary

Support virtual test suites in wptrunner, so that we can run tests with
additional browser flags to exercise different code paths.

## Details

Add an *optional* command line argument to pass in a json configuration file. The
content of the configuration file is defined by each vendor, and defines a list of
virtual test suites. Each virtual test suite has a prefix, and a list of bases
and args. For example, you could define a (hypothetical) virtual test suite
using the following json configuration:

```json
{
  "prefix": "css-highlight",
  "bases": ["css/css-pseudo", "css/css-highlight-api"],
  "args": ["--enable-features=css-highlight"]
}
```

This will create new "virtual" tests of the form
`virtual/css-highlight/css/css-pseudo/...` and
`virtual/css-highlight/css/css-highlight-api/...` which correspond to the files
under `css/css-pseudo` and `css/css-highlight-api`, respectively,
and pass `--enable-features=css-highlight` to the browser when they are run.

### Managing test expectations

Virtual tests will have their own expectation files (.ini) at
virtual/prefix/..., and fall back to its non-virtual variants.

## Risks

It would be pretty easy to add a huge virtual test suite, and subsequently cause
a test bloat. It is up to each vendor to control the size of their virtual test
suites.

## Standarlized virtual test suites

We consider virtual test suites are mostly vendor specific. There could be a
need to compare virtual test results accoss browsers. In such case we need to
use the same prefix for all browsers. We consider this topic out of scope of
this RFC, but we need to reserve a name space for standarlized virtual test
suites if we can foretell there will be such use cases.
