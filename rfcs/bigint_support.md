# RFC 222: Support BigInts in numerical assertion functions and output

## Summary

Allow BigInt values (e.g. `1234n`) in `assert_less_than()` and friends.


## Details

https://github.com/web-platform-tests/wpt/issues/51331

Background: BigInts are a relatively new fundamental numeric type in JavaScript that allow for arbitrary size integers. They are commonly used for 64-bit signed and unsigned integers (e.g. in conjunction with `BigInt64Array` and `BigUint64Array`) but are not limited to that range. Literals are written with an `n` suffix (e.g. `let my_num = 1234n;`, and the `BigInt()` function can be used to "cast" another value. They interoperate transparently with regular JavaScript Numbers in many cases, but are a distinct type: `typeof 1234n === 'bigint'`.

The following assertion functions will be changed to support passing BigInts as the expected and actual inputs:

* `assert_less_than()`
* `assert_greater_than()`
* `assert_less_than_equal()`
* `assert_greater_than_equal()`
* `assert_between_exclusive()`
* `assert_between_inclusive()`

Specifically:
* The functions are updated to allow either Number or BigInts to be passed, instead of failing if anything other than a Number is passed.
* For the `between` functions, the upper and lower bounds must be the same type. The test itself is considered at fault if the types don't match.
* The assertions fail if the type of the actual value does not match the type of the expected value.

Assertion functions for "approximate" comparisons (`assert_approx_equals()` and `assert_array_approx_equals()`) that take an epsilon value will *not* be modified by this RFC.

In addition, the internal `format_value()` used when formatting messages such as assertion failures will be updated to output BigInt values with the `n` suffix, e.g. `1234n`. Currently it switches on the `typeof` the value and falls through to the default case, which outputs `bigint "1234"`.

WIP implementation PR: https://github.com/web-platform-tests/wpt/pull/51919


## Risks

Any change to `testharness.js` can affect error stacks. This could cause a mismatch in "expectation" files maintained by implementations and failures in their CIs, requiring baseline updates. This is probably unavoidable maintenance burden for such tests. Here is one example in a prototype update, in the Chromium repository: https://chromium-review.googlesource.com/c/chromium/src/+/6351358/6/third_party/blink/web_tests/external/wpt/html/infrastructure/safe-passing-of-structured-data/structured-cloning-error-stack-optional.sub.window-expected.txt

The change to `format_value()` affects the output of any existing WPTs that output BigInts. This could cause a mismatch in "expectation" files maintained by implementations and failures in their CIs, requiring baseline updates. From inspection, there are no obvious cases.

Requiring the types of the expected and actual values to match could potentially confuse test authors, as JavaScript allows writing `123n > 456`.
