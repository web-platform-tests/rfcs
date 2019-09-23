# `testharness.js`: Replace `assert_throws` and `promise_rejects` with more fine-grained APIs

## Summary

The expected value passed to `assert_throws` and `promise_rejects` APIs can be one of several different kinds of values.
This makes it harder to correctly compare the actual thrown exception / rejection value.
Remove these APIs, and replace them with more focused APIs.

## Details

The `assert_throws` and `promise_rejects` APIs verify the value that is thrown or passed as a rejection value.
They support being called with three kinds of expected values:

- a string or number representing a `DOMException` (e.g., `"INVALID_STATE_ERR"`, `"InvalidStateError"`, or `DOMException.INVALID_STATE_ERR`);
- an ECMAScript-style `Error` object (e.g., `new TypeError()`);
- a plain object (in practice, the exact same object should be thrown or used as the rejection value).

The last two are handled the same way: only the "name" properties are compared between the expected and the actual value.
Because of the conflation of the various cases, the existing APIs don't get any of them quite right, and this causes some browser bugs to be missed.
For example, the following passes:

```js
assert_throws(new TypeError(), () => { throw new DOMException("TypeError") });
```

Six new APIs are proposed to handle each of the cases separately:

- `promise_rejects_dom` and `assert_throws_dom`, called with a string representing a `DOMException` (as above);
- `promise_rejects_js` and `assert_throws_js`, called with the Error constructor (e.g. `TypeError`);
- `assert_throws_exactly` and `promise_rejects_exactly`, called with the expected value (which no longer needs to be an object) and using `Object.is`.

The new APIs will be added first, in order to allow a well-organized transition.
Once this is done and browser vendors have updated their copies of `testharness.js`, we can start working on a (mostly automated) conversion of the tests in wpt.
The old API will only be removed once all callers have been removed from wpt and browser vendors have had a chance to update any callers in proprietary tests.

In wpt, there are approximately 4200 calls to `assert_throws` (of which approximately 1200 in the generated canvas tests) and approximately 1400 to `promise_rejects`.

## Risks

Updates to testharness.js haven't always been made together with the test updates in all browser engine repos.
If this is still the case, the testharness.js changes have to be made first and synced downstream before any test changes are made.

## References

This was discussed in [wpt issue #11726](https://github.com/web-platform-tests/wpt/issues/11726) and the implementation of the new APIs was done in [wpt PR #19054](https://github.com/web-platform-tests/wpt/pull/19054).
