# RFC 81: Add testdriver.js support for forcing JS Self-Profiler sampling

## Summary

Add testdriver.js support for the `Force Sample` WebDriver command, which enables deterministic testing of the [JS Self-Profiling API](https://wicg.github.io/js-self-profiling/). This command is defined in the [JS Self-Profiling API spec](https://wicg.github.io/js-self-profiling/#force-sample).

## Details

The JS Self-Profiling API exposes a periodic sampling profiler to JavaScript for measuring CPU execution. The spec itself does not normatively require samples to be taken at the user's desired interval, as sampling periodicity is often at the whim of the device's scheduler.

To enable normative testing of the API, as well as reduce flakiness, this RFC proposes defining a function `test_driver.force_sample()` to invoke the [Force Sample WebDriver extension command](https://wicg.github.io/js-self-profiling/#force-sample).

An example usage in a test might look like:

```
promise_test(() => {
  const profiler = new Profiler({ sampleInterval: 10 });

  (function foo() {
    test_driver.force_sample();
  })();

  const trace = await profiler.stop();
  verifyTraceContainsFunction(trace, 'foo');
}, 'verify that function expression names are captured');
```

## Risks

- It's possible that implementation behaviour may differ between calls to `force_sample` and periodic sampling.
  - UAs must take care to ensure that `force_sample`'s implementation shares code with normal, periodic sampling.
- As with other RFCs, this extends the API surface of testdriver, and introduces a nonzero maintenance burden.
