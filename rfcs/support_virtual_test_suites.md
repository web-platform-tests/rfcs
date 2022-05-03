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

## Implementations

### Integration with Manifests

Virtual tests are built on top of real tests, and will have their own test
identities by prepending 'virtual/<prefix>/' to the identities of the
corresponding real tests. There is no need to add virtual tests into
Manifest.json as that will unnecessarily increase the size of the file. Instead we
can load virtual tests after the real tests are loaded from Manifest.json.

### Running virtual tests

As virtual tests have their own test identities, virtual tests can be sharded the
same way as the real tests. When running the test, we will need to check if the
running browser instance is started with the desired command line arguments. We
need to restart the browser if the command line arguments do not match. We should
arrange the virtual tests properly to minimize the possibility that a restart is
needed.

When sending the url to browser, we should strip 'virtual/<prefix>/' from the
test url so that the browser can load the correct resources for testing.

### Managing test expectations

Virtual tests can have their own expectation files (.ini) at
virtual/<prefix>/..., and fall back to its non-virtual variants.

### Test reports

Test reports can be generated the same way as before, as virtual tests have
their own test identities. Test results for virtual tests generally should not be
published on wpt.fyi, as virtual tests are deemed vendor specific.

## Risks

It would be pretty easy to add a huge virtual test suite, and subsequently cause
a test bloat. It is up to each vendor to control the size of their virtual test
suites.
