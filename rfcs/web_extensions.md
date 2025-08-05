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
The documentation for API can be viewed [here](https://github.com/w3c/webextensions/blob/main/proposals/browser_test_api.md).

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
