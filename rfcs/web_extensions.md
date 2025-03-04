# RFC #219: Add support for WebExtensions in WPT

## Summary

The WebExtensions API extends the capabilities of the browser. Adding support
for WebExtensions in the web platform tests will increase interoperability
and will help drive the standardization of this API.

This RFC proposes adding a new `testharness.js` test type, `.extension.js`, to handle
testing this API, in addition to using `testdriver.js` to load and unload extensions.
   
##  Details
  
Using `testdriver.js`, we added support for testing the WebExtensions API by loading
a web extension designed to test the functionality of a specific API.
The extension will be loaded after the tests begins, and unloaded before the
test is finished.

Most of the test execution is handled within the extension, via the 
[browser.test](https://chromium.googlesource.com/chromium/src.git/+/master/extensions/docs/testing_api.md)
API. We’ve elected to use these APIs since all participating browser vendors use
`browser.test`  internally and they can easily port over existing tests to
the web platform tests.

Because these tests won’t leverage `testharness.js` directly, we’ve introduced a new
`testharness.js`, `.extension.js`, that will create the necessary boilerplate to
convert the `browser.test` assertions into the corresponding assertions in the test
harness.
   
A proposed patch is available at https://github.com/web-platform-tests/wpt/pull/50648.

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
   behavior to be the same. The Classic implementation is defined in the WebExtensions Community Group
   [here](https://github.com/w3c/webextensions/blob/main/specification/webdriver-classic.bs),
   and the BiDi implementation is defined
   [here](https://www.w3.org/TR/webdriver-bidi/#module-webExtension).

2. Another concern could be with using `browser.test` assertions and mapping them to
   `testharness.js` assertions. With `testharness.js` not in charge of generating assertions,
   we might end up with less useful failure messages.
