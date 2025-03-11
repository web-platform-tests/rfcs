# RFC #219: Add support for WebExtensions in WPT

## Summary

The WebExtensions API extends the capabilities of the browser. Adding support
for WebExtensions in the web-platform-tests will increase interoperability
and will help drive the standardization of this API.

This RFC proposes adding a new `testharness.js` wrapper type, `.extension.js`, to handle
testing this API, in addition to using `testdriver.js` to load and unload extensions.
   
##  Details
  
Using `testdriver.js`, we added support for testing the WebExtensions API by loading
a web extension designed to test the functionality of a specific API.

These tests won't leverage `testharness.js` directly since most of the test execution
is handled within the extension, via the `browser.test` API. We’ve elected to use
this API since all participating browser vendors use `browser.test` internally and
can easily port over existing tests to the web-platform-tests.  

The `browser.test` API will also be used to send messages back to the web page to notify the testharness
when a test has started and finished. For these cases, it is up to the user agent to send the message
"test-started" and "test-finished", respectively. In addition, when recording the result for 
a `browser.test.assert...()` method, it is up to the user agent to send the message "assert"
or "assert-equality". 

Because these tests don’t leverage `testharness.js` directly, we’ve introduced a new
wrapper type, `.extension.js`, that will create the necessary boilerplate to
convert these messages into the corresponding assertions in the `testharness.js`.
   
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
Asserts that the actual value is equal to the expected value. If not equal, the test fails.

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
**`browser.test.runTests(tests)`**\
Queues an array of test functions to run and returns a promise.
The promise is rejected if one of the tests fails, otherwise resolves.

**Parameters**

* `tests` (array) –  An array of functions.

\
**`browser.test.onMessage(listener)`**\
Event for receiving messages sent by the test harness. 

**Parameters**

* `listener` (function) –  A function that gets called when a message is received.


## Alternatives considered

We considered loading the extension statically before each `testharness.js` test is run,
but we decided against that since `testdriver.js` will allow the tests to drive the
browser, and in the future, test functionality such as opening popups and clicking
menu items. 

We considered adding a new test type rather than using JavaScript tests, but we decided
against it because it was a lot more work compared to using `testharness.js`.
   
## Risks

There are two potential concerns with this implementation:

1. We have no precedent for tests run via a Classic command in some user agents
   and BiDi command in others. However, the Classic implementation of loading
   and unloading extension is modeled on the BiDi implementation, so we expect the
   behavior to be the same. The Classic implementation is defined in the 
   [WECG](https://github.com/w3c/webextensions/blob/main/specification/webdriver-classic.bs), and the BiDi implementation is defined in [WebDriver Bidi](https://www.w3.org/TR/webdriver-bidi/#module-webExtension).

2. Another concern could be with using `browser.test` assertions and mapping them to
   `testharness.js` assertions. With `testharness.js` not in charge of generating assertions,
   we might end up with less useful failure messages.
