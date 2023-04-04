# RFC 141: Change testharness to include late tests

## Summary

Currently, "late tests" are
[ignored](https://github.com/web-platform-tests/wpt/blob/c751476c04678faa79e598683f2a6e94d17cf9e0/resources/test/tests/unit/late-test.html#L10-L17).
This RFC proposes that testharness should now include them.

## Details

Conside the following test:

```
\<script>
window.onload = () => {
  test((t) => { ... }, 'test 1');
  test((t) => { ... }, 'test 2');
  test((t) => { ... }, 'test 3');
};
\</script>
```

Currently, only test 1 will be run. The issue is that the testharness
immediately adds a window load handler that marks all_loaded = true,
and that ends the tests as soon as the first result from the first test
is processed. (The test runner waits for the first test because
Tests.prototype.all_done() also waits until this.tests.length > 0.)
There were various mitigating corner cases, such as if you started
the list of tests with a promise_test(), that would increment a
counter that kept the rest of the tests alive. Etc.

The proposal is to change testharness window.onload handler to run
with a setTimeout(0) so that all_loaded is only set to true after all of
the tests are loaded by any window.onload handler.

## Risks

This change will expose a few tests that should have been failing but were
masked by the lack of test coverage - bugs have been filed for those.

Also, several tests that were working around this via various
means will be cleaned up in the implementation of this RFC. 
