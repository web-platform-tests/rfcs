# RFC 178: Signaling when preconditions are not met on a reftest

## Summary

This is a proposal to add a way to make a reftest neither fail nor pass, but return `PRECONDITION_FAILED`. This is useful if some precondition of the normative statement we want to test is not satisfied.

## Details

Sometimes, a specification will make a normative statement about something that MUST be done if a certain precondition is fulfilled. If the precondition is not fulfilled, there's nothing to test. For testharness, there is [assert_implements_optional](http://web-platform-tests.org/writing-tests/testharness-api.html#assert_implements_optional), but for reftests, there is no equivalent.

This is particularly annoying when the precondition is en environmental factor over which the test has no control.

* "When printing, the UA must…", but you're running the test on screen, not printing
* "On devices with a keyboard, the UA must…", but you're running the test on a phone
* "If the UA uses overlay scrollbars, the UA must…", but you're running the test on a device with old-fashioned layout-affecting scrollbars

Note: I'm not talking about the case where you cannot detect whether the precondition has been fulfilled or not, but rather of the case where the test can detect that it is not fulfilled.

From a logical point of you, you could set up all these tests to pass, since the UA has violated no normative requirement. However, this would give a false sense of comfort: for example, UAs that have not even implemented the feature at all would pass the test when you run them in the right (i.e. wrong) environment.

It'd be a lot more accurate to return `PRECONDITION_FAILED`, rather than passed or failed. So far, there's no way to signal that.

## Proposed solution: use a waiting reftest and add a class on the root

If a reftest is setup with `class="reftest-wait"` on the root, the harness will wait before taking a screenshot until that class has been removed. We can take advantage of that by running tests in js during that time, and signaling through another class on the root when the precondition has failed. For instance `class="precondition-failed"`

We could add a convenience method , similar to `takeScreenshot()` to `/common/reftest-wait.js`:

```js
function preconditionFailed() {
    document.documentElement.classList.add("precondition-failed");
    document.documentElement.classList.remove("reftest-wait");
}
```

This would be picked up by the test runner, which would then report the `PRECONDITION_FAILED` statust, rather than a failed or passed one.

### Risks

Tests that return `PRECONDITION_FAILED` cannot be included in interop stats because it's not clear how they should be counted.
