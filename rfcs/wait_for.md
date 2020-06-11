# RFC 56: `step_wait*` methods

## Summary

Add `step_wait`, `step_wait_func` and `step_wait_func_done` methods to
the wpt `Test` object that evaluate a condition, and resolve a
promise, or call a callback, once the condition is true.

## Details

Using timers in tests is problematic; depending on the speed of the
system the test can either wait a long time unnecessarily or be flaky
because the operation sometimes takes longer than allowed by the
timeout. The timeout multiplier in conjunction with `step_timeout`
can alleviate this by adjusting the timeout to the overall system
speed, but authors are still incentivised to provide short timeouts in
order to allow tests to run faster.

In some cases it's possible to detect directly whether the thing being
waited on has occured or not. In this case the performance/robustness
tradeoff can be improved by having a relatively long total timeout but
polling for success to allow the test to proceed as soon as possible.

To encourage this pattern we introduce three new methods on `Test`:

 * `step_wait_func(cond, callback, description, timeout=3000,
 interval=100)`
 * `step_wait_func_done(cond, callback, description, timeout=3000,
interval=100)`
 * `step_wait(cond, description, timeout=3000, interval=100)`

In each case the function `cond` is called with no
arguments, immediately and then every `interval` milliseonds until it
returns a value that evaluates as `true`. At this point `callback` is called.
If this does not happen in `timeout` milliseconds, an `AssertionError` is
raised, using `description` as the error description.

The difference between the functions is in what happens after `cond`
evaluates to true:

 * `step_wait_func` - `callback` is called as a step on `Test`.
 * `step_wait_func_done` - `callback` is called as a step on `Test` if
   it's not `undefined` or `null`. After `callback` returns, or in any
   case if it's not called, the `done` function on test is called.
 * `step_wait` - The initial function call returns a promise, which is
   resolved when `cond` evaluates to true.

These functions must methods on `Test` to allow any error raised to be
associated with the correct subtest.

The default timeout and interval were adopted from the
`waitForCondition`
[function](https://searchfox.org/mozilla-central/source/testing/mochitest/tests/SimpleTest/SimpleTest.js#1236)
in Mozilla's mochitest, which implements a similar pattern, and has
been used to support a policy of only allowing `setTimeout` with
non-zero timeout in exceptional cases.

## Examples

A test which waits to check that `contentDocument` on an `iframe`
becomes null after some action, might intially look like:

```
    t.step_timeout(() => {
      assert_equals(frame.contentDocument, null);
      t.done();
    }, 2000);
```

With these APIs it can instead look like

```
    t.step_wait_func_done(() => frame.contentDocument === null);
```

## Risks

* Adding API surface to testharness.js requires long-term support or
  is quite difficult to remove or change if the feature is deemed to
  be a failure.
* Features like this and `step_timeout` could be useful in support
  files, but using testharness.js features from support files is
  tricky.
* This RFC doesn't propose a lint to flag uses of `step_timeout` /
  `setTimeout`, so it's hard to educate authors about the optimal
  pattern. Such a lint could be proposed as a followup, but our
  linting infrastructure lacks support for parsing js code, so it's
  hard to enforce complex conditions like only allowing zero-duration
  timeouts.
