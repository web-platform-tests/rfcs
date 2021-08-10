# RFC 93: Allow testharness.js to depend on Promise being available

## Summary

Currently testharness.js has a test-enforced policy that ES Promises
are only required by features which specifically depend on promises
(e.g. `promise_test`). Other functionality must work in the absence of
promises.

This RFC removes that restriction, allowing ES Promises to be used
throughout the testharness.js codebase.

## Details

Historically ES Promises were not widely supported across browser
engines. Native promises became available in 2014, and even some years
later browsers that were known to make use of web-platform-tests were
running JavaScript engine versions without Promise support. In
particular Servo was making heavy use of web-platform-tests but didn't
support promises until late
[2016](https://github.com/servo/servo/issues/4282), whilst Internet
Explorer has never had Promise support.

Due to the Servo requirements in particular, the testharness.js tests
were written to run in a mode with the [global promise constructor
removed](https://github.com/web-platform-tests/wpt/commit/08995880734011eed359c4e79420db29626016ff#diff-d33a74992c12a398b4876be7ea865df0387f779f740be2ff79e792f7d4db5badR12-R14). This
ensures that features which don't explicitly depend on Promise don't
accidentally rely on it, allowing most tests to run in
implementations without Promises.

Today the number of browsers which might conceivably run wpt but lack
Promise support is very low. Many new DOM APIs require promises, so
the only implementations which lack Promise support are those such as
IE 11 which are no longer receiving feature updates. This makes it
safe to rely on promises across the entire testharness codebase. It
opens up the possibility of refactorings to make the code simpler, and
removes a point of confusion for new contributors who are unaware of
the unusual policy of avoiding promises.

The proposal is that we allow promises throughout testharness as a
matter of policy, and remove the variants feature from the
testharness.js tests, along with all usage of it. Since this feature
is currently only used for with and without promise variants, we can
simplify test authoring by not having the additional complexity.

## Risks

* Some unusual older browsers might still be running wpt. For example
  we know that the WAVE server is used for testing TV-based browsers,
  which could conceivably be so outdated as to not support
  promises. We don't have much insight into how this is used so could
  conceivably result in "dark matter" breakage.

* Removing the variants feature from testharness.js tests entirely
  might be premature if we find we need similar functionality in the
  future. However the fact that we've currently only used it for this
  one use case, and the fact that it adds noticeable complexity to test
  authoring and to the harness suggests we should consider re-adding
  the feature if we find an additional use case.
