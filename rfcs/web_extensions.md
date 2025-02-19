# RFC #219: Add support for WebExtensions in WPT

## Summary

The WebExtensions API extends the capabilities of the browser. Adding support for WebExtensions in
the web-platform-tests will increase interoperability and will help drive the standardization of
this API.

This RFC proposes adding a new `testharness.js` wrapper type, `.extension.js`, to handle testing this
API, in addition to using `testdriver.js` to load and unload extensions. A new `feature=extensions`
was also introduced to support enabling `BiDi` specifcally for user agents who use it for the
web-platform-tests. This feature can be removed once all browser vendors use `BiDi` for these tests.


##  Details

Using `testdriver.js`, we added support for testing the WebExtensions API by loading
a web extension designed to test the functionality of a specific API.

These tests won't leverage `testharness.js` directly since most of the test execution is handled
within the extension, via the `browser.test` API. We’ve elected to use this API since all participating
browser vendors use `browser.test` internally and can easily port over existing tests to the
web-platform-tests.

The `browser.test` API will also be used to send messages back to the web page to notify the
testharness when a test has started and finished. For these cases, it is up to the user agent to
send the message "test-started" and "test-finished", respectively.

Because these tests don’t leverage `testharness.js` directly, we’ve introduced a new wrapper type,
`.extension.js`, that will create the necessary boilerplate to convert these messages into the
corresponding assertions in the `testharness.js`.

It's important to note that it isn't guaranteed that the web page will load before the extension loads.
Meaning, a race condition could occur that would cause the `browser.test` assertion results to not
be reported back to the test harness, since the `browser.test.onMessage` listener wasn't registered.
To resolve this, user agents would need to queue the results until a listener is registered, and then
report them.

Last, Since the `Classic` and `BiDi` implementations support loading an extension using a `path` or
`archivePath` to the extension's resources, the full path to the extension's resources is formulated
by using the `test.path` before the protocol loads the extension.

A proposed patch is available at https://github.com/web-platform-tests/wpt/pull/50648.

##  `browser.test` API

The `browser.test` API provides utilities for writing and running tests for WebExtensions.
It includes assertion functions and a test harness to facilitate structured and reliable testing.


### Methods

**`browser.test.assertTrue(condition, message)`**\
Asserts that a given condition is `true`. If the condition is `false`,
the test fails.

**Parameters**

* `condition` (boolean) – The condition to assert. If `false`, the test fails.
* `message` (string, optional) – A message describing the assertion.

\
**`browser.test.assertFalse(condition, message)`**\
Asserts that a given condition is `false`. If the condition is `true`, the test fails.

**Parameters**

* `condition` (boolean) – The condition to assert. If `true`, the test fails.
* `message` (string, optional) – A message describing the assertion.

\
**`browser.test.assertEq(actual, expected, message)`**\
Throws an assertion error and fails the test if the actual value is not _logically_ equal to the
expected value, by these rules:

1) _Plain arrays_ (`Array.isArray()` returns true) are compared for _deep_ equality.
    1.1) Their `length` and all items are recursively compared by these same rules.
2) _Plain objects_ (with prototype `Object.prototype` or `null`) are also compared recursively.
    2.1) Only enumerable _own_ properties are considered, unordered.
3) Everything else is compared using `Object.is()` [same value algorithm](https://tc39.es/ecma262/#sec-samevalue).

We're starting with support for the minimal set of compound types, and plan to expand custom rules
to include `ArrayBuffer`s, `Map`s, `Set`s, etc. in congruence with Node's [assert.deepStrictEqual()](https://nodejs.org/api/assert.html#comparison-details_1).

**Parameters**

* `actual` (any) – The actual value.
* `expected` (any) – The expted value.
* `message` (string, optional) – A message describing the assertion.

\
**`browser.test.assertThrows(fn, message)`**\
Asserts that a given function throws an error when executed. If the function does not throw,
the test fails.

**Parameters**

* `fn` (function) – A function expected to throw an error.
* `message` (string, optional) – A message describing the assertion.

\
**`browser.test.runTests(tests)`**
Queues an array of test functions to be executed sequentially and returns a promise that resolves or
rejects based on the test outcomes.

Each function in the `tests` array is treated as an individual test case. Tests are executed in the
order they appear in the array. If multiple tests are passed, a new test begins once the previous
tests completes. If a test fails—either due to a failed assertion or an unexpected exception—execution
of that test stops, and the next test in the queue begins.

After the returned promise is settled, this method can be called again to run additional tests.
Calling it while test are still running throws or returns a rejected promise.

The returned promise:
- **Resolves** if all tests complete successfully.
- **Rejects** if any test fails.

A test is considered **complete** when:
- It returns a non-promise value (e.g., `undefined`) for synchronous tests, or
- It returns a resolved or rejected promise for asynchronous tests.

A test is considered **passed** if:
- It returns `undefined` (for synchronous tests), or
- It returns a resolved promise (for asynchronous tests).

A test is considered **failed** if:
- It throws an exception,
- It returns a rejected promise, or
- It triggers an assertion failure via the `browser.test.assert...` methods.

Uncaught exceptions and unhandled promise rejections that occur during the execution of a test
are not treated as test failures.

**Parameters**

* `tests` (array) –  An array of functions.

\
**`browser.test.onMessage(listener)`**\
Event for receiving messages sent by the test harness.

**Parameters**

* `listener` (function) –  A function that gets called when a message is received.

Messages the browser agents can send to the test harness, and the expected data parameters:

**"test-started" `data` parameters**

* `testName` (string) – The name of the test being run.

**"test-finished" `data` parameters**

* `testName` (string) – The name of the test that ran.
* `result` (boolean) – A boolean indicating whether the test passed or failed.
* `remainingTests` (number) – The number of remaining tests to run.
* `message` (string, optional) – An additional message provided by the test assertion.
* `assertionDescription` (string) – A human-readable description, such as "Assertion Failed: Expected: 'string'; Actual: 'function'".

## Alternatives considered

We considered loading the extension statically before each `testharness.js` test is run, but we
decided against that since `testdriver.js` will allow the tests to drive the browser, and in the
future, test functionality such as opening popups and clicking menu items.

We considered adding a new test type rather than using JavaScript tests, but we decided against it
because it was a lot more work compared to using `testharness.js`.

Lastly, since Chrome uses the `BiDi` implementation, we considered using the existing `feature=bidi`
flag to enable `BiDi` for the tests. However, since not all browser vendors support `BiDi`, using
this flag could be problematic since this feature would be enabled for tests that wouldn't always
need `BiDi` enabled. To resolve this, a new `feature=extensions` was added, which as of now, enables
`BiDi` for Chrome.


## Risks

There are two potential concerns with this implementation:

1. We have no precedent for tests run via a Classic command in some user agents and BiDi command in
   others. However, the Classic implementation of loading and unloading extension is modeled on the
   BiDi implementation, so we expect the behavior to be the same. The Classic implementation is
   defined in the  [WECG](https://github.com/w3c/webextensions/blob/main/specification/webdriver-classic.bs), and the BiDi implementation is defined in [WebDriver BiDi](https://www.w3.org/TR/webdriver-bidi/#module-webExtension).

2. Another concern could be with using `browser.test` assertions and mapping them to
   `testharness.js` assertions. With `testharness.js` not in charge of generating assertions,
   we might end up with less useful failure messages.
