# RFC 56: `wait_for[_callback]` methods

## Summary

Add `wait_for` and `wait_for_callback` methods to the wpt `Test`
object that evaluate a condition, and resolve a promise, or call a
callback, respectively, once the condition is true.

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

To encourage this pattern we introduce two new methods on `Test`:

`wait_for_callback(cond, callback, description, timeout=3000,
interval=100)` and `wait_for(cond, description, timeout=3000,
interval=100)`. In each case the function `cond` is called with no
arguments, immediately and then every `interval` milliseonds until it
returns a value that evaluates as `true`. At this point `callback` is called.
If this does not happen in `timeout` milliseconds, an `AssertionError` is
raised, using `description` as the error description.

`wait_for(cond, description, timeout=3000, interval=100)` works like
`wait_for_callback` except it returns a promise that is resolved once the function
`cond` returns true.

These functions must be methods on `Test` to allow any error that's
raised to be associated with the correct subtest.

The default timeout and interval were adopted from the
`waitForCondition`
[function](https://searchfox.org/mozilla-central/source/testing/mochitest/tests/SimpleTest/SimpleTest.js#1236)
in Mozilla's mochitest, which implements a similar pattern, and has
been used to support a policy of only allowing `setTimeout` with
non-zero timeout in exceptional cases.

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
  hard to enforce complex conditions.
