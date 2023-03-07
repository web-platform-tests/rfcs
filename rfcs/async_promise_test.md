# RFC 75: async_promise_test()

## Summary

Since `await`/`async` have been introduced into the language, many tests have
switched to it. Tests are now easier to read and write. Events are happening in
the same order as they are written in the test. No more callback to follow.

`promise_tests()` instead of `async_tests()`/`step_func()`/... So far, so good!
However this comes at a cost, `promise_tests()` are running in sequence,
`async_tests()` in parallel. Running too many `promise_tests()` causes timeout.
Developers are forced to split tests into many files, or switch back to
`async_tests()` against their will. 

We would like a parallel version. This could be part of `testharness.js` or be
put inside an helper script.

There would be 4 kind of test. Promise or function based. Running in sequence or
in parallel:

|              | in sequence | in parallel       |
| -------------| ----------: | ----------------: |
| **Function** | test        | async_test        |
| **Promise**  | promise_test| async_promise_test|

## Details

Some tests are defining it as:
```javascript
// Test using the modern async/await primitives are easier to read/write.
// However they run sequentially, contrary to async_test. This is the parallel
// version, to avoid timing out.
let async_promise_test = (promise, description) => {
  async_test(test => {
    promise(test)
    .then(() => {test.done();})
    .catch(test.step_func(error => { throw error; }));
  }, description);
};
```

## Risks

This is a pure addition. The main risk is bad design and not being able to
improve it later, if used massively.

In particular, we should think more about how this will interact with other
functions from `test_harness.js`, like `promise_setup()`.
