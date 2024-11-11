# RFC XXX: Add `?feature` query parameter to `testdriver.js`

## Summary

This RFC is an alternative to [RFC 212](https://github.com/web-platform-tests/rfcs/pull/212). It proposes adding a query parameter `?features` to `testdriver.js`. At the moment the only supported feature will be `bidi`, which will be used to enable the [testdriver BiDi API](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/testdriver_bidi.md) for specific WPT tests, The supported features can be extended by other RFCs if required. The feature `bidi`, applicable to HTML and JS files, would let test runners choose between [**WebDriverProtocol**](https://github.com/web-platform-tests/wpt/blob/9c76757f5332678f9952f6ccb3824f62d30eca1f/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L644) and [**WebDriverBidiProtocol**](https://github.com/web-platform-tests/wpt/blob/9c76757f5332678f9952f6ccb3824f62d30eca1f/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L736). This approach does not require all the browsers to implement WebDriver BiDi.

[Prototype](https://github.com/web-platform-tests/wpt/pull/48622).

## Background

[RFC 185](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/testdriver_bidi.md) proposes adding BiDi support to  `testdriver.js`, which involves using [WebDriverBidiProtocol](https://github.com/web-platform-tests/wpt/blob/9c76757f5332678f9952f6ccb3824f62d30eca1f/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L736) instead of [WebDriverProtocol](https://github.com/web-platform-tests/wpt/blob/9c76757f5332678f9952f6ccb3824f62d30eca1f/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L644) for test runners that use the WebDriver protocol. However, this change could cause regressions in tests that don't use the testdriver BiDi API due to unintended side effects of the transport change. To address this, the query parameter `?features=bidi` in `testdriver.js` will inform the test runner whether the given test requires the testdriver BiDi API, allowing the runner to choose the appropriate transport, if the default one does not support BiDi protocol parts.

If at a certain point all the browsers implement testdriver BiDi and enable it by default, this flag can be deprecated.

## Details

### Examples

[html test](https://github.com/web-platform-tests/wpt/blob/9395d384f5c69a9a3a7fc4de04249f77500b2d3f/infrastructure/webdriver/bidi/subscription.html#L3)
```javascript
<!DOCTYPE html>
<meta charset="utf-8">
<title>Test console log are present</title>
<script src="/resources/testharness.js"></script>
<script src="/resources/testharnessreport.js"></script>
<script src="/resources/testdriver.js?features=bidi"></script>
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
// META: script=/resources/testdriver.js?features=bidi

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

#### Features

The `testrdriver_features` is a list of values of query parameters with the name `feature`, used when the `testdriver.js` is imported. It has to be passed from `SourceFile` through `TestharnessTest` to `Test`, where it can be consumed by the specific test runners.

`testdriver.js` will provide or not the `bidi` domain based on having or not the `feature=bidi` query parameter when it was included.

The `feature` query parameter can be set when `testdriver.js` is included either in `.html` or `.js`. The [SourceFile](https://github.com/web-platform-tests/wpt/blob/9395d384f5c69a9a3a7fc4de04249f77500b2d3f/tools/manifest/sourcefile.py#L503) collects the features in the `testrdriver_features` list and passes it to the [TestharnessTest](https://github.com/web-platform-tests/wpt/blob/9395d384f5c69a9a3a7fc4de04249f77500b2d3f/tools/manifest/item.py#L169) object. The `TestharnessTest` object then passes the `testrdriver_features` property to the `Test` object.

The test runner can use the `Test`’s `testrdriver_features` property to determine whether to enable BiDi support for the test.

Even though the `TestExecutor` needs information about whether to activate BiDi or not at `connect`, which happens before the runner receives the test to run, the configuration can be done at the point [the browser is configured](https://github.com/web-platform-tests/wpt/pull/48618/files#diff-0df24b5b583c460182e687f7dc7a6a79dd2cd3389bc4a96f48483f60fceb51f7R272). This data can be used later by the specific executor implementation to [decide which protocol to use](https://github.com/web-platform-tests/wpt/pull/48618/files#diff-c2f15328bc1ddfa8fb93c09ae41651e2bcfdad4f257932263a3e92c3f8deffceR248).

#### Lint rules

Some lint rules has to be added to verify is used correctly:
* It is unique per test file.
* It has the correct value (either `true` or `false`).

### Deprecating of the `require_webdriver_bidi` metadata tag

Expectations are that eventually all the implementations would support BiDi either via WebDriverBiDi, or via implementation-specific bindings. At that point, the `require_webdriver_bidi` metadata tag can be deprecated. This can be done by simply removing the tag.

## Risks

### Having metadata in the Test is not sufficient

`TestExecutor` needs information about whether to activate BiDi or not at `connect`, which happens before the runner receives the test metadata. This can be addressed by passing the data to implementation-specific `TestharnessExecutor` via the implementation-specific `BrowserSettings` and  like in the [example](https://github.com/web-platform-tests/wpt/pull/48618/files#diff-0df24b5b583c460182e687f7dc7a6a79dd2cd3389bc4a96f48483f60fceb51f7R269).

## Alternatives considered

List of alternatives is in the [RFC 212](https://github.com/web-platform-tests/rfcs/pull/212).
