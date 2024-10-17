# RFC 212: Introduce `require_webdriver_bidi` metadata tag

## Summary

To facilitate the gradual enablement of WebDriver BiDi transports, this RFC proposes the introduction of a new metadata tag, `require_webdriver_bidi` in html and js files in WPT tests. This metadata would allow for a gradual transition from [WebDriverProtocol](https://github.com/web-platform-tests/wpt/blob/9c76757f5332678f9952f6ccb3824f62d30eca1f/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L644) to [WebDriverBidiProtocol](https://github.com/web-platform-tests/wpt/blob/9c76757f5332678f9952f6ccb3824f62d30eca1f/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L736), ensuring that implementations can opt-in into WebDriver BiDi gradually in case there are implementation blockers like in Chrome's case. [Prototype](https://github.com/web-platform-tests/wpt/pull/48622).

## Background

[RFC 185: Add WebDriver BiDi support to testdriver.js](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/testdriver_bidi.md) discusses incorporating BiDi functionality into `testdriver.js`. This entails migrating from [WebDriverProtocol](https://github.com/web-platform-tests/wpt/blob/9c76757f5332678f9952f6ccb3824f62d30eca1f/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L644) to [WebDriverBidiProtocol](https://github.com/web-platform-tests/wpt/blob/9c76757f5332678f9952f6ccb3824f62d30eca1f/tools/wptrunner/wptrunner/executors/executorwebdriver.py#L736). However, this switch may cause regression in certain tests that don't utilize the testdriver BiDi API due to unintended side effects resulting from the transport change. Nonetheless, all these regressions need to be resolved eventually. Implementing a flag will enable immediate provision of the testdriver BiDi API to tests, eliminating the need for test authors to wait for the migration of all previous tests to `WebDriverBidiProtocol`.

## Details

### Examples

[html test](https://github.com/web-platform-tests/wpt/blob/9395d384f5c69a9a3a7fc4de04249f77500b2d3f/infrastructure/webdriver/bidi/subscription.html#L3)
```javascript
<!DOCTYPE html>
<meta charset="utf-8">
<meta name="require_webdriver_bidi" content="true" />
<title>Test console log are present</title>
<script src="/resources/testharness.js"></script>
<script src="/resources/testharnessreport.js"></script>
<script src="/resources/testdriver.js"></script>
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
// META: require_webdriver_bidi=true
// META: script=/resources/testdriver.js

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

The `require_webdriver_bidi` property has to be passed from `SourceFile` through `TestharnessTest` to `Test`, where it can be consumed by specific test runners.

The `require_webdriver_bidi` property is a flag that indicates that the test requires testdriver BiDi API support. This property is used by the test runner to determine which tests to run and how to configure the test session. Note that not having that flag does not require implementation to restrict BiDi.

The `require_webdriver_bidi` property can be set in the test source file, either `.html` or `.js`. The [SourceFile](https://github.com/web-platform-tests/wpt/blob/9395d384f5c69a9a3a7fc4de04249f77500b2d3f/tools/manifest/sourcefile.py#L503) object reads that property and passes it to the [TestharnessTest](https://github.com/web-platform-tests/wpt/blob/9395d384f5c69a9a3a7fc4de04249f77500b2d3f/tools/manifest/item.py#L169) object. The `TestharnessTest` object then passes the `require_webdriver_bidi` property to the `Test` object.

The test runner can use the `Test`’s `require_webdriver_bidi` property to determine whether to enable BiDi support for the test. If the `require_webdriver_bidi` property is set to `true`, the test runner will create a session with BiDi support.

Even though the `TestExecutor` needs information about whether to activate BiDi or not at `connect`, which happens before the runner receives the test to run, the configuration can be done at the point [the browser is configured](https://github.com/web-platform-tests/wpt/pull/48618/files#diff-0df24b5b583c460182e687f7dc7a6a79dd2cd3389bc4a96f48483f60fceb51f7R272). This data can be used later by the specific executor implementation to [decide which protocol to use](https://github.com/web-platform-tests/wpt/pull/48618/files#diff-c2f15328bc1ddfa8fb93c09ae41651e2bcfdad4f257932263a3e92c3f8deffceR248).

#### Alternatives

* [Add metadata to `.ini` files](#add-metadata-to-ini-files).

#### Lint rules

Some lint rules has to be added to verify is used correctly:
* It is unique per test file.
* It has the correct value (either `true` or `false`).

### Deprecating of the `require_webdriver_bidi` metadata tag

Expectations are that eventually all the implementations would support BiDi either via WebDriverBiDi, or via implementation-specific bindings. At that point, the `require_webdriver_bidi` metadata tag can be deprecated. This can be done by simply removing the tag.

## Risks

### Having metadata in the Test is not sufficient

`TestExecutor` needs information about whether to activate BiDi or not at `connect`, which happens before the runner receives the test metadata. This can be addressed by passing the data to implementation-specific `TestharnessExecutor` via the implementation-specific `BrowserSettings` and  like in the [example](https://github.com/web-platform-tests/wpt/pull/48618/files#diff-0df24b5b583c460182e687f7dc7a6a79dd2cd3389bc4a96f48483f60fceb51f7R269).

### Potential false-negative due to missing `require_webdriver_bidi` tag

Some implementations can enable BiDi by default, while others would still need enabling it via `require_webdriver_bidi`. This could lead to false-negative test passes by the tests requiring BiDi but not having the `require_webdriver_bidi` tag. This risk can be mitigated by adding documentation describing the logic behind.

## Alternatives considered

### Add metadata to `.ini` files

In this alternative approach, the `require_webdriver_bidi` metadata would be added to the corresponding `.ini` file. However, this approach was ultimately declined because the `.ini` file metadata is primarily concerned with the expectations of the test run rather than the specific API requirements. On the other hand, the `require_webdriver_bidi` metadata is directly related to the test API itself.

### Extract testdriver BiDi API into a separate file

In this alternative, the testdriver BiDi API is extracted from testdriver. Include of the file is used as a marker to enable BiDi sessions. This alternative involves unnecessary indirections, and would require non-trivial tracking of nested includes. Separating the testdriver BiDi API from the main codebase adds complexity and maintenance challenges. It also requires careful tracking of nested includes to ensure accurate BiDi session enabling. A simpler implementation should be considered.
