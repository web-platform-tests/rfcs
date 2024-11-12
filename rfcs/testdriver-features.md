# RFC 214: Add testdriver features

## Summary

This RFC is an alternative to [RFC 212](https://github.com/web-platform-tests/rfcs/pull/212) providing more generic and extendable solution. It proposes adding a query parameter `?feature` to `testdriver.js` and adding a `testdriver_features` property to the [TestharnessTest](https://github.com/web-platform-tests/wpt/pull/49122/files#diff-8f280b64b0700ab9b4b343adabc7ff4d5ea4b1f45cb6c4bd7f50a19b21ebdb8cR169). At the moment the only supported feature will be `bidi`, which will be used to enable the [testdriver BiDi API](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/testdriver_bidi.md) for specific WPT tests. The feature `bidi`, applicable to HTML and JS files, would let test runners choose between [**WebDriverProtocol**](https://github.com/web-platform-tests/wpt/pull/49122/files#diff-c2f15328bc1ddfa8fb93c09ae41651e2bcfdad4f257932263a3e92c3f8deffceR277) and [**WebDriverBidiProtocol**](https://github.com/web-platform-tests/wpt/pull/49122/files#diff-c2f15328bc1ddfa8fb93c09ae41651e2bcfdad4f257932263a3e92c3f8deffceR279). This approach does not require all the browsers to implement WebDriver BiDi.

This approach is preferable over the [RFC 212](https://github.com/web-platform-tests/rfcs/pull/212), as it is more ergonomic and supports future extensions ergonomically. The list of supported features can be extended when required.

[Prototype](https://github.com/web-platform-tests/wpt/pull/49122).

## Background

[RFC 185](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/testdriver_bidi.md) proposes adding BiDi support to  `testdriver.js`, which involves using [**WebDriverProtocol**](https://github.com/web-platform-tests/wpt/blob/f54cd920024da0857fdc5b036f16a7d1fd8792fd/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L587) instead of [**WebDriverBidiProtocol**](https://github.com/web-platform-tests/wpt/blob/f54cd920024da0857fdc5b036f16a7d1fd8792fd/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L679) for test runners that use the WebDriver protocol. However, this change could cause regressions in tests that don't use the testdriver BiDi API due to unintended side effects of the transport change. This RFC proposes a solution for the issue.

## Details

### Examples

[html test](https://github.com/web-platform-tests/wpt/blob/9395d384f5c69a9a3a7fc4de04249f77500b2d3f/infrastructure/webdriver/bidi/subscription.html#L3)
```javascript
<!DOCTYPE html>
<meta charset="utf-8">
<title>Test console log are present</title>
<script src="/resources/testharness.js"></script>
<script src="/resources/testharnessreport.js"></script>
<script src="/resources/testdriver.js?feature=bidi"></script>
<script src="/resources/testdriver-vendor.js"></script>
<script>
    promise_test(async () => {
        const some_message = "SOME MESSAGE";
        // Subscribe to `log.entryAdded` BiDi events. This will not add a listener to the page.
        await test_driver.bidi.log.entry_added.subscribe();
        // Add a listener for the log.entryAdded event. This will not subscribe to the event, so the subscription is
        // required before. The cleanup is done automatically after the test is finished.
        const log_entry_promise = test_driver.bidi.log.entry_added.once();
        // Emit a console.log message.
        // Note: Lint rule is disabled in `lint.ignore` file.
        console.log(some_message);
        // Wait for the log.entryAdded event to be received.
        const event = await log_entry_promise;
        // Assert the log.entryAdded event has the expected message.
        assert_equals(event.args.length, 1);
        const event_message = event.args[0];
        assert_equals(event_message.value, some_message);
    }, "Assert testdriver can subscribe and receive events");
```

```javascript
// META: script=/resources/testdriver.js?feature=bidi

'use strict';

promise_test(async () => {
    const some_message = "SOME MESSAGE";
     // Subscribe to `log.entryAdded` BiDi events. This will not add a listener to the page.
    await test_driver.bidi.log.entry_added.subscribe();
    // Add a listener for the log.entryAdded event. This will not subscribe to the event, so the subscription is
    // required before. The cleanup is done automatically after the test is finished.
    const log_entry_promise = test_driver.bidi.log.entry_added.once();
    // Emit a console.log message.
    // Note: Lint rule is disabled in `lint.ignore` file.
    console.log(some_message);
    // Wait for the log.entryAdded event to be received.
    const event = await log_entry_promise;
    // Assert the log.entryAdded event has the expected message.
    assert_equals(event.args.length, 1);
    const event_message = event.args[0];
    assert_equals(event_message.value, some_message);
}, "Assert testdriver can subscribe and receive events");
```

### Required changes

#### Configure testdriver.js based on the ?feature flag

The `testdriver.js` checks if the `?feature=bidi` query parameter was included using [document.currentScript.src](https://developer.mozilla.org/en-US/docs/Web/API/Document/currentScript) like in the [prototype](https://github.com/web-platform-tests/wpt/pull/49122/files#diff-1fe2b624679a3150e5c86f84682c5901b715dad750096a524e8cb23939e5590fR16). If `bidi` was not enabled, a [descriptive error will be thrown](https://github.com/web-platform-tests/wpt/pull/49122/files#diff-1fe2b624679a3150e5c86f84682c5901b715dad750096a524e8cb23939e5590fR22).

#### Add `testrdriver_features` to TestharnessTest

This RFC proposes adding `testrdriver_features` to [`TestharnessTest`](https://github.com/web-platform-tests/wpt/pull/49122/files#diff-8f280b64b0700ab9b4b343adabc7ff4d5ea4b1f45cb6c4bd7f50a19b21ebdb8cR169) so that executors can get the list of features declared by a test. The list is populated by parsing the `feature` query param from the `testdriver.js` imports used by a test. Currently, only the `bidi` feature is supported.

#### (implementation-specific) Set up WebDriver BiDi in the test executor if needed

Even though the `TestExecutor` needs information about whether to activate BiDi or not at `connect`, which happens before the runner receives the test to run, the configuration can be done at the point [the browser is configured](https://github.com/web-platform-tests/wpt/pull/49122/files#diff-0df24b5b583c460182e687f7dc7a6a79dd2cd3389bc4a96f48483f60fceb51f7R275). This data can be used later by the specific executor implementation to [decide which protocol to use](https://github.com/web-platform-tests/wpt/pull/49122/files#diff-0df24b5b583c460182e687f7dc7a6a79dd2cd3389bc4a96f48483f60fceb51f7R275).

## Risks

### Having `testrdriver_features` in the `Test` is not sufficient

`TestExecutor` needs information about whether to activate BiDi or not at `connect`, which happens before the runner receives the test metadata. This can be addressed by passing the data to implementation-specific `TestharnessExecutor` via the implementation-specific `BrowserSettings` and  like in the [example](https://github.com/web-platform-tests/wpt/pull/49122/files#diff-0df24b5b583c460182e687f7dc7a6a79dd2cd3389bc4a96f48483f60fceb51f7R269).

## Alternatives considered

List of other alternatives is in the [RFC 212](https://github.com/web-platform-tests/rfcs/pull/212).

### Always provide `bidi` API in testdriver

In this alternative, the testdriver does not raise an error if the bidi feature was not enabled.This creates a risk of potential false-negative test results, if the test author forgets to add the `feature=bidi` and check the test only on the browser implementation which supports WebDriver BiDi by-default, regardless of the feature.
