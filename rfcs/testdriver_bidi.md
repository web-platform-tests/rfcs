# RFC 182: Allow WebDriver BiDi in testdriver.js
## Summary
Add “testdriver.js” support for [WebDriver BiDi](https://w3c.github.io/webdriver-bidi/) events and actions. To support WebDriver BiDi, the following parts are required:
* **Test_runner support for asynchronous actions.** WebDriver BiDi commands are asynchronous. To implement them, the test runner must support asynchronous actions. It should support concurrent actions as well.
* **Test_runner - test_driver transport support for events.** WebDriver BiDi events can be sent from the browser to the client. In order to support WebDriver BiDi events, test_runner should be able to send events to test_driver.
* **Extend test_driver API.** test_driver should be extended to support WebDriver BiDi commands and events.
* **Update RFC policy.** (optional, can be done in a different RFC) In [RFC policy](https://github.com/web-platform-tests/rfcs/blob/master/README.md) do not require RFC for changes extending “testdriver.js” with a method that closely matches a [WebDriver BiDi](https://w3c.github.io/webdriver-bidi) protocol.
## Background
The “testdriver.js” framework (or test_driver for simplicity) is based on the one-directional WebDriver Classic protocol and lacks the ability to receive events from the browser via test_runner, limiting our ability to test event-based functionalities such as console events in WPT (e.g. many [console log tests](https://github.com/web-platform-tests/wpt/tree/master/console) are [manual](https://github.com/web-platform-tests/wpt/blob/master/console/console-timing-logging-manual.html)!). To overcome this limitation, the [WebDriver BiDi](https://w3c.github.io/webdriver-bidi) protocol is proposed as an extension for WebDriver Classic, offering bidirectional communication between the test and the browser.
This document presents changes in test_driver and test_runner so that it incorporates both WebDriver Classic and WebDriver BiDi functionalities, demonstrating the addition of tests for console events as an [example](https://github.com/web-platform-tests/wpt/pull/44649).
## Details
### Glossary
* test_driver: “testdriver.js” framework used by tests and providing a way to communicate with browser via test_runner.
* test_runner: backend python scripts implementing test harness, providing communication between test_driver and browser.
* test_harness: communication channel between test_driver and test_runner.
### Example
Here is an [example](https://github.com/sadym-chromium/wpt/blob/596b2d3f39116b805db22ae350c7341ab47f7601/console/console-log-logged.html) of a test using the proposed `test_driver.bidi` API for testing console log events.
```javascript
promise_test(async () => {
    const some_message = "SOME MESSAGE";
    // Enable log.entryAdded BiDi events.
    await test_driver.bidi.log.entry_added.subscribe();
    // Add a listener for the log.entryAdded event.
    const log_entry_promise = test_driver.bidi.log.entry_added.once();
    // Emit a console.log message.
    console.log(some_message);
    // Wait for the log.entryAdded event to be received.
    const event = await log_entry_promise;
    // Assert the log.entryAdded event has the expected message.
    assert_equals(event.args.length, 1);
    const event_message = event.args[0];
    assert_equals(event_message.value, some_message);
    await test_driver.bidi.log.entry_added.unsubscribe();
}, "Assert log event is logged");
```
### Required changes
The proposal changes the following parts:
#### test_driver
The test_driver requires support of event processing, and exposing it as an API.
##### test_driver API
Introduce a new namespace `bidi` to the `test_driver`, and to `test_driver_internal`. The namespaces will be divided into separate WebDriver BiDi modules like in the [example](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-1fe2b624679a3150e5c86f84682c5901b715dad750096a524e8cb23939e5590fR54): `test_driver.bidi.{module}.{event/action}`.
###### Actions
The BiDi actions are functionally equivalent to the classic actions. They are placed in the `test_driver.bidi.{module}.{action}`. In order to prevent action name clashes with the classic ones, action names (those used to communicate to the test_runner) should have a form of `bidi.{module}.{action}`.
####### Alternative
[Flatten bidi and classic actions](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.9apnhuz21k2).
###### Events
The exposed events API in `test_driver` provides the following methods for each event:
* `test_driver.bidi.{module}.{event}.subscribe(): Promise<void>`. The async method subscribes to the given WebDriver BiDi event, but does not add event listeners yet.
* `test_driver.bidi.{module}.{event}.unsubscribe(): Promise<void>`. The async method unsubscribes from the given WebDriver BiDi event.
* `test_driver.bidi.{module}.{event}.on(handler: (event)=>void): ()=>void`. The method makes the handler be called each time the event emitted. The callback argument is the event params. It returns a handle for removing this listener.
* (optional) `test_driver.bidi.{module}.{event}.once(): Promise<event>`. Returns a promise, which will be resolved after the required event is emitted for the first time, and removes the listener. This method is not required, but allows easier writing of some tests (e.g. it is used in the example above). It can be easily done via the `on` method.
  ####### Alternative
  [Make event subscription generic](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.tjaczmn1wj1h).
##### Event processing
The proposal extends the event processing implemented in [`testdriver-extra.js`](https://github.com/web-platform-tests/wpt/blob/master/tools/wptrunner/wptrunner/testdriver-extra.js) with a new message type [“testdriver-event”](https://github.com/sadym-chromium/wpt/blob/50a730c8e1b3c34d1e201d6d700abfbebc587998/tools/wptrunner/wptrunner/testdriver-extra.js#L43). This message is sent by the test_runner to notify the test_driver about new events. It uses the `message` field to store JSON-serialized `params` and `method`. The `testdriver-extra.js` overrides `window.test_driver_internal.bidi.{modules}.{event}` object methods `subscribe`, `unsubscribe`, and `on`.
#### test_runner
Currently, the test_runner processes communication with the test_driver in a blocking way, which makes it impossible to process events from the browser. To overcome this limitation, support of coroutines should be done.
##### `AsyncCallbackHandler`
Add a `AsyncCallbackHandler`, which is an extension of `CallbackHandler` with support for async actions. It requires an event loop, in which it will schedule the tasks. Async action processing is done by overriding the `_send_action_response` method. If the action result is a coroutine, schedule an action response message to test_driver after the action result coroutine is done and return immediately. [Example](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-3de439d55203876d452599a1a14968409df69e5869b16918ceec378a6b3345a4R794) implementation.
##### `WebDriverProtocol`
As long as WebDriver Classic and WebDriver BiDi share the session, the `WebDriverProtocol` should be able to set a `enable_bidi` flag when creating the session. Extend the class with the  `enable_bidi=false` flag, which will be overridden by `WebDriverBidiProtocol`.
##### `WebDriverBidiProtocol`
###### Dependency on `WebDriverProtocol`
New `WebDriverBidiProtocol` should be introduced. It is an extension of WebDriverProtocol, but with additional BiDi-specific protocol parts, e.g. `WebDriverEventsProtocolPart` and `WebDriverBidiBaseProtocolPart`. In order to enable the bidi session, the inherited `enable_bidi` flag should be overridden to true. [Example](https://github.com/sadym-chromium/wpt/pull/448/files#diff-e5a8911dd97e0352b1b26d8ce6ef0a92b25378f7c9e79371a1eb1b1834bc9a8dR470).
###### Dedicated loop
To be able to process events from the browser and from the tests in the same time, the WebDriverBidiProtocol protocol should have a dedicated [event loop](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-e5a8911dd97e0352b1b26d8ce6ef0a92b25378f7c9e79371a1eb1b1834bc9a8dR578), which will be used by test_runner to run async actions and process events in WebDriver BiDi transport.
###### Alternative
[Add `AsyncProtocolPart` with `get_loop` method](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.76ejnn78ir5m).
##### `TestharnessExecutor`
In this example we consider changes in `WebDriverTestharnessExecutor`.  Other `TestharnessExecutor` (for now it is `MarionetteTestharnessExecutor`, `SeleniumTestharnessExecutor`, `ServoWebDriverTestharnessExecutor`,
`WKTRTestharnessExecutor`) implementations should be done in the same way in order to support WebDriver BiDi.
###### Messages from test_driver to test_runner
The communication channel between test_driver and test_runner is set in the `do_testharness` method. Currently the messages from test_driver to test_runner it is done via long poll by calling BaseProtocolPart.execute_async_script which is synchronous (async in the name indicates that the script is wrapped in a special way before sent to the browser), and blocks processing the event  loop in test_runner, which in turn blocks the event processing while waiting for the next message form test_driver to test_runner. To overcome this limitation, this RFC changes the test_runner - test_driver communication channel to asynchronous non-blocking  `BidiScriptProtocolPart.async_call_function` if available.  `WebDriverBidiProtocol`’s event loop should be used for message processing. [Example](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-e5a8911dd97e0352b1b26d8ce6ef0a92b25378f7c9e79371a1eb1b1834bc9a8dR725).
###### Processing asynchronous actions
If `WebDriverBidiProtocol` is available, test_runner uses `AsyncCallbackHandler` with the `WebDriverBidiProtocol`’s event loop instead of the sync `CallbackHandler`.
###### Forwarding events from test_runner to test_driver
If `BidiEventsProtocolPart` is available, test_runner sets a callback via `add_event_listener` and forwards all the events to the test_driver. [Example](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-e5a8911dd97e0352b1b26d8ce6ef0a92b25378f7c9e79371a1eb1b1834bc9a8dR695).
###### Alternative
[Add `async_execute_script` to `BaseProtocolPart`](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.d7fexpinxvnd).
##### `BidiEventsProtocolPart`
To provide events from test_runner to test_driver, the `BidiEventsProtocolPart` implements add_event_listener. `TestharnessExecutor` will subscribe and forward events to test_driver.
##### `BidiScriptProtocolPart`
`BidiScriptProtocolPart` is required for implementing asynchronous poll messages from test_driver.
## Risks
### Future changes in WebDriver BiDi protocol
The WebDriver BiDi protocol may still undergo specification changes that could impact WPT tests.
#### Mitigation
* **Add commands on-demand.** To reduce the effects of future changes to the protocol, commands can be added as and when required instead of preemptively including all possible commands in the test_driver API.
* **Keep the test_driver API minimally required.** By keeping the test_driver API as minimal as possible, the impact of future protocol changes can be further reduced, as fewer API changes will be required.
### WebDriver BiDi implementation differences
Different implementations of WebDriver BiDi may have variations that could potentially complicate its use within the context of the WPT tests.
#### Mitigation
Ensure the test_driver API is compliant with the specification by avoiding reliance on implementation-specific data and behavior.
### WebDriver BiDi events might not agree with some observable side effects
In certain tests, the observable side effects may not correspond entirely with the WebDriver BiDi events. One such instance arises when a console.log BiDi event occurs, which does not guarantee the automatic display of the console message in the browser's developer tools.
#### Mitigation
This risk has a low priority impact and should not impede the project's progress.
### Different WebDriver BiDi implementation status
The implementation of WebDriver BiDi may vary across different endpoints, which can lead to tests utilizing BiDi functionality failing for endpoints lacking the required test components.
#### Mitigation
Expanding test_driver with WebDriver BiDi shouldn't cause previously passing tests to fail. This is because any regressions would only occur due to new tests that utilize WebDriver BiDi encountering incompatible endpoints - a scenario explicitly excluded from regression consideration.
### TODO: any other risks?
TODO
## Alternatives considered
The list of alternatives of different granularity levels:
### Flatten bidi and classic actions in `test_driver`
In this alternative, in [test_driver actions](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.1tdetff8j5en), the new WebDriver BiDi methods are added directly to the root test_driver object instead of introducing a dedicated `bidi` namespace. This approach has both advantages and disadvantages:
#### Advantages
No need in introducing a new namespace, and continue using the established pattern of adding new methods.
#### Disadvantages
* Potential for name clashes: If a new WebDriver BiDi action or event has the same name as an existing WebDriver Classic action or event, it could cause conflicts.
* Harder to determine if a specific test_driver method requires WebDriver BiDi support: Developers may need to consult the documentation or code to determine if a particular method requires WebDriver BiDi support.
### Make event subscription generic
In this alternative, `test_driver` exposes an `EventTarget`, or methods `addEventListener(type, listener)`, and `removeEventListener(type, listener)`. Test will be able to subscribe to any WebDriver BiDi event.
#### Advantages
This approach is more flexible, as there will be no need in adding support for each WebDriver BiDi event.
#### Disadvantages
The API would depend on the WebDriver BiDi, and writing / reading tests would require being familiar with it.
### Do not expose `session.subscribe`, do it silently in the `on`method
In this alternative, the `test_driver.session.subscribe` method is not exposed in the test_driver API. Instead, each event `on` handler has to subscribe to the required events.
#### Advantages
The API becomes simpler. Test writers would not require care about the concept of WebDriver BiDi subscriptions at all.
#### Disadvantages
The `once` handler will not be possible to implement.
### Do not expose `session.subscribe`, expose event’s  `enable` method
In this alternative, the `test_driver.session.subscribe` method is not exposed in the test_driver API. Instead, each event `on` handler has to subscribe to the required events.
#### Advantages
The API is easier to use, as all the subscription logic is hidden from the user in the specific events.
#### Disadvantages
Less flexible. Each new event will require not only adding `on`, but `enable` as well.
### Add async_execute_script to BaseProtocolPart
In this alternative, the `BaseProtocolPart` should be extended with the async `async_execute_script` method. The alternative is declined, as it requires significant changes in the WebDriver Classic client.
### Add `AsyncProtocolPart` with `get_loop` method
Instead of exposing the event loop in the WebDriverBidiProtocol, add a dedicated `AsyncProtocolPart` protocol part, which exposes the loop via `get_loop` method. The alternative is declined, as the event loop is not a part of the protocol, but rather the implementation detail.
