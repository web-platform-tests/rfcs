# RFC #219: Add support for WebExtensions in WPT

## Summary

The WebExtensions API extends browser capabilities. Adding support for WebExtensions in
web-platform-tests increases interoperability and helps drive the standardization of this API.

This RFC proposes adding a new `testharness.js` wrapper type, `.extension.js`, to handle testing
this API, in addition to using `testdriver.js` to load and unload extensions. A new
`feature=extensions` is also introduced to support enabling `BiDi` specifically for user agents
that use it for web-platform-tests. This feature can be removed once all browser vendors use
`BiDi` for these tests.

## Details

Using `testdriver.js`, we support testing the WebExtensions API by loading a web extension
designed to test the functionality of a specific API.

These tests don’t leverage `testharness.js` directly since most of the test execution happens
within the extension via the `browser.test` API. We use this API because all participating
browser vendors use `browser.test` internally and can easily port existing tests over to
web-platform-tests.

The `browser.test` API also sends messages back to the web page to notify the testharness when
a test starts and finishes. In these cases, the user agent fires the `browser.test.onTestStarted`
and `browser.test.onTestFinished` event listeners, respectively.

Because these tests don’t leverage `testharness.js` directly, we introduce a new wrapper type,
`.extension.js`, that creates the necessary boilerplate to convert these messages into the
corresponding assertions in `testharness.js`.

It’s important to note that it isn’t guaranteed that the web page loads before the extension.
A race condition could occur that causes the `browser.test` assertion results not to be reported
back to the test harness if the `browser.test.onMessage` listener isn’t registered. To resolve
this, user agents queue the results until a listener is registered, then report them.

Lastly, since the `Classic` and `BiDi` implementations support loading an extension using a
`path` or `archivePath` to the extension’s resources, the full path to the extension’s resources
is formulated using `test.path` before the protocol loads the extension.

A proposed patch is available at https://github.com/web-platform-tests/wpt/pull/50648.

## `browser.test` API

The `browser.test` API provides utilities for writing and running tests for WebExtensions. It
includes assertion functions and a test harness to facilitate structured and reliable testing.

### Methods

**`browser.test.assertTrue(condition, message)`**
Asserts that a given condition is `true`. If the condition is `false`, the test fails.

**Parameters**
- `condition` (boolean) – The condition to assert.
- `message` (string, optional) – A message describing the assertion.

**`browser.test.assertFalse(condition, message)`**
Asserts that a given condition is `false`. If the condition is `true`, the test fails.

**Parameters**
- `condition` (boolean)
- `message` (string, optional)

**`browser.test.assertEq(actual, expected, message)`**
Throws an assertion error and fails the test if the actual value is not logically equal to the
expected value. Rules:
1) _Plain arrays_ (`Array.isArray()` returns true) are compared for _deep_ equality.
    1.1) Their `length` and all items are recursively compared by these same rules.
2) _Plain objects_ (with prototype `Object.prototype` or `null`) are also compared recursively.
    2.1) Only enumerable _own_ properties are considered, unordered.
3) Everything else is compared using `Object.is()` [same value algorithm](https://tc39.es/ecma262/#sec-samevalue).

We start with support for a minimal set of compound types and plan to expand to include
`ArrayBuffer`s, `Map`s, `Set`s, etc., in line with Node’s `assert.deepStrictEqual()`.

**Parameters**
- `actual` (any)
- `expected` (any)
- `message` (string, optional)

**`browser.test.assertThrows(fn, expectedError, message)`**
Asserts that a given function throws an error. If it does not, or if the thrown error doesn't
match `expectedError` (if defined), the test fails.

**Parameters**
- `fn` (function)
- `expectedError` (string | RegExp, optional)
- `message` (string, optional)

**`browser.test.runTests(tests)`**
Queues test functions to run sequentially and returns a promise that resolves or rejects based
on the outcome.

Each test function is treated as an individual test case. Tests run in order. When one completes,
the next begins. If a test fails (by throwing or rejecting), execution of that test stops and the
next one begins.

The promise:
- **Resolves** if all tests pass.
- **Rejects** if any fail.

Tests pass when they return `undefined` or a resolved promise. They fail if they:
- Throw an exception
- Return a rejected promise
- Trigger an assertion failure

**Parameters**
- `tests` (array of functions)

**`browser.test.onTestStarted(listener)`**
Event listener that runs when a test starts.

**Parameters**
- `listener` (function)

**Listener receives:**
- `data.testName` (string)

**`browser.test.onTestFinished(listener)`**
Event listener that runs when a test finishes.

**Parameters**
- `listener` (function)

**Listener receives:**
- `data.testName` (string)
- `result` (boolean)
- `remainingTests` (number)
- `message` (string, optional)
- `assertionDescription` (string)

## Alternatives Considered

We considered loading the extension statically before each `testharness.js` test runs, but chose
against it. `testdriver.js` allows tests to drive the browser, which enables us to test
behaviors such as opening popups and clicking menu items.

We also considered defining a new test type, but this would introduce much more complexity
compared to using `testharness.js`.

Finally, since Chrome uses `BiDi`, we considered using `feature=bidi` to enable it. However, since not
all vendors support it, we introduce `feature=extensions` instead, which currently enables `BiDi`
for Chrome only.

## Risks

Two concerns arise:

1. We have no precedent for tests that run via a Classic command in some agents and a BiDi
command in others. However, the Classic implementation is modeled after BiDi, so we expect
consistent behavior. Classic is defined in [WECG](https://github.com/w3c/webextensions/blob/main/specification/webdriver-classic.bs),
and BiDi in [WebDriver BiDi](https://www.w3.org/TR/webdriver-bidi/#module-webExtension).

2. Mapping `browser.test` assertions to `testharness.js` may result in less useful failure
messages, since `testharness.js` no longer generates them directly.
