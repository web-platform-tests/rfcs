# RFC 182: Add WebDriver BiDi support to testdriver.js
## Summary
Add “testdriver.js” support for [WebDriver BiDi](https://w3c.github.io/webdriver-bidi/) events and actions. To support WebDriver BiDi, the following parts are required:
* **Testrunner support for asynchronous actions.** WebDriver BiDi commands are asynchronous. To implement them, the test runner must support asynchronous actions. It should support concurrent actions as well.
* **Testrunner - testdriver transport support for events.** WebDriver BiDi events can be sent from the browser to the client. In order to support WebDriver BiDi events, testrunner should be able to send events to testdriver.
* **Extend testdriver API.** testdriver should be extended to support WebDriver BiDi commands and events.
* **Update RFC policy.** (optional, can be done in a different RFC) In [RFC policy](https://github.com/web-platform-tests/rfcs/blob/master/README.md) do not require RFC for changes extending “testdriver.js” with a method that closely matches a [WebDriver BiDi](https://w3c.github.io/webdriver-bidi) protocol.
## Background
The “testdriver.js” framework (or testdriver for simplicity) is based on the one-directional WebDriver Classic protocol and lacks the ability to receive events from the browser via testrunner, limiting our ability to test event-based functionalities such as console events in WPT (e.g. many [console log tests](https://github.com/web-platform-tests/wpt/tree/master/console) are [manual](https://github.com/web-platform-tests/wpt/blob/master/console/console-timing-logging-manual.html)!). To overcome this limitation, the [WebDriver BiDi](https://w3c.github.io/webdriver-bidi) protocol is proposed as an extension for WebDriver Classic, offering bidirectional communication between the test and the browser.
This document presents changes in testdriver and testrunner so that it incorporates both WebDriver Classic and WebDriver BiDi functionalities, demonstrating the addition of tests for console events as an [example](https://github.com/web-platform-tests/wpt/pull/44649).
## Details
### Glossary
* testdriver: “testdriver.js” framework used by tests and providing a way to communicate with the browser via testrunner.
* testrunner: backend python scripts implementing test harness, providing communication between testdriver and browser.
* test harness: communication channel between testdriver and testrunner.
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
#### Testdriver
The testdriver requires support of event processing, and exposing it as an API.
##### Testdriver API
Introduce a new namespace `bidi` to the `test_driver`, and to `test_driver_internal`. The namespaces will be divided into separate WebDriver BiDi modules like in the [example](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-1fe2b624679a3150e5c86f84682c5901b715dad750096a524e8cb23939e5590fR54): `test_driver.bidi.{module}.{event/action}`.
###### Actions
The BiDi actions are functionally equivalent to the classic actions. They are placed in the `test_driver.bidi.{module}.{action}`. In order to prevent action name clashes with the classic ones, action names (those used to communicate to the testrunner) should have a form of `bidi.{module}.{action}`.
####### Alternative
[Flatten bidi and classic actions in `test_driver`](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.9apnhuz21k2).
###### Events
The exposed events API in `test_driver` provides the following methods for each event:
* `test_driver.bidi.{module}.{event}.subscribe(): Promise<void>`. The async method subscribes to the given WebDriver BiDi event, but does not add event listeners yet.
* `test_driver.bidi.{module}.{event}.unsubscribe(): Promise<void>`. The async method unsubscribes from the given WebDriver BiDi event.
* `test_driver.bidi.{module}.{event}.on(handler: (event)=>void): ()=>void`. The method makes the handler be called each time the event emitted. The callback argument is the event params. It returns a handle for removing this listener.
* (optional) `test_driver.bidi.{module}.{event}.once(): Promise<event>`. Returns a promise, which will be resolved after the required event is emitted for the first time, and removes the listener. This method is not required, but allows easier writing of some tests (e.g. it is used in the example above). It can be easily done via the `on` method.
  ####### Alternative
* [Make event emitting generic in testdriver](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.tjaczmn1wj1h).
* [Do not expose `test_driver.bidi.{module}.{event}.subscribe()`, do it silently in the `on` method](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.19pmlwcnuau3)
##### Event processing
The proposal extends the event processing implemented in [`testdriver-extra.js`](https://github.com/web-platform-tests/wpt/blob/master/tools/wptrunner/wptrunner/testdriver-extra.js) with a new message type [“testdriver-event”](https://github.com/sadym-chromium/wpt/blob/50a730c8e1b3c34d1e201d6d700abfbebc587998/tools/wptrunner/wptrunner/testdriver-extra.js#L43). This message is sent by the testrunner to notify the testdriver about new events. It uses the `message` field to store JSON-serialized `params` and `method`. The `testdriver-extra.js` overrides `window.test_driver_internal.bidi.{modules}.{event}` object methods `subscribe`, `unsubscribe`, and `on`.
#### Testrunner
Currently, the testrunner processes communication with the testdriver in a blocking way, which makes it impossible to process events from the browser. To overcome this limitation, support of coroutines should be done.
##### Introduce `async_actions`
Analogous to `actions`, introduce an async_actions which will support asynchronous implementations (e.g. [`BidiSessionSubscribeAction`](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-9d6d3e870d033b8bc4d75b388fcc11dcf332e0293c286b2d57734c74e22020f7R4)). These async actions will be processed by `AsyncCallbackHandler`.
##### Alternative
[Extend `CallbackHandler` to support async actions](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.gi63kouttvhv).
##### Alternative
[Extend testdriver - testrunner transport with `async_action`](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.djn1ih2znce1).
##### Introduce `AsyncCallbackHandler`
Add a `AsyncCallbackHandler`, which is an extension of `CallbackHandler` with support for async actions. It requires an event loop, in which it will schedule the tasks. Async action processing is done in the same way as CallbackHandler is done, but in the dedicated task. This would not allow raising unexpected exceptions. [Example](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-3de439d55203876d452599a1a14968409df69e5869b16918ceec378a6b3345a4R813) implementation.
##### Extend `WebDriverProtocol`
As long as WebDriver Classic and WebDriver BiDi share the session, the `WebDriverProtocol` should be able to set a `enable_bidi` flag when creating the session. Extend the class with the  `enable_bidi=false` flag, which will be overridden by `WebDriverBidiProtocol`.
##### Introduce `WebDriverBidiProtocol`
###### Dependency on `WebDriverProtocol`
New `WebDriverBidiProtocol` should be introduced. It is an extension of WebDriverProtocol, but with additional BiDi-specific protocol parts, e.g. `WebDriverEventsProtocolPart` and `WebDriverBidiBaseProtocolPart`. In order to enable the bidi session, the inherited `enable_bidi` flag should be overridden to true. [Example](https://github.com/sadym-chromium/wpt/pull/448/files#diff-e5a8911dd97e0352b1b26d8ce6ef0a92b25378f7c9e79371a1eb1b1834bc9a8dR470).
###### Dedicated loop
To be able to process events from the browser and from the tests in the same time, the WebDriverBidiProtocol protocol should have a dedicated [event loop](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-e5a8911dd97e0352b1b26d8ce6ef0a92b25378f7c9e79371a1eb1b1834bc9a8dR578), which will be used by testrunner to run async actions and process events in WebDriver BiDi transport.
###### Alternative
[Add `AsyncProtocolPart` with `get_loop` method](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.76ejnn78ir5m).
##### Extend `TestharnessExecutor`
In this example we consider changes in `WebDriverTestharnessExecutor`.  Other `TestharnessExecutor` (for now it is `MarionetteTestharnessExecutor`, `SeleniumTestharnessExecutor`, `ServoWebDriverTestharnessExecutor`,
`WKTRTestharnessExecutor`) implementations should be done in the same way in order to support WebDriver BiDi.
###### Messages from testdriver to testrunner
The communication channel between testdriver and testrunner is set in the `do_testharness` method. Currently the messages from testdriver to testrunner it is done via long poll by calling BaseProtocolPart.execute_async_script which is synchronous (async in the name indicates that the script is wrapped in a special way before sent to the browser), and blocks processing the event  loop in testrunner, which in turn blocks the event processing while waiting for the next message form testdriver to testrunner. To overcome this limitation, this RFC changes the testrunner - testdriver communication channel to asynchronous non-blocking  `BidiScriptProtocolPart.async_call_function` if available.  `WebDriverBidiProtocol`’s event loop should be used for message processing. [Example](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-e5a8911dd97e0352b1b26d8ce6ef0a92b25378f7c9e79371a1eb1b1834bc9a8dR725).
###### Processing asynchronous actions
If `WebDriverBidiProtocol` is available, testrunner uses `AsyncCallbackHandler` with the `WebDriverBidiProtocol`’s event loop instead of the sync `CallbackHandler`.
###### Forwarding events from testrunner to testdriver
If `BidiEventsProtocolPart` is available, testrunner sets a callback via `add_event_listener` and forwards all the events to the testdriver. [Example](https://github.com/web-platform-tests/wpt/pull/44649/files#diff-e5a8911dd97e0352b1b26d8ce6ef0a92b25378f7c9e79371a1eb1b1834bc9a8dR695).
###### Alternative
[Add `async_execute_script` to `BaseProtocolPart`](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.d7fexpinxvnd).
##### Introduce `BidiEventsProtocolPart`
To provide events from testrunner to testdriver, the `BidiEventsProtocolPart` implements add_event_listener. `TestharnessExecutor` will subscribe and forward events to testdriver.
##### Introduce `BidiScriptProtocolPart`
`BidiScriptProtocolPart` is required for implementing asynchronous poll messages from testdriver.
## Risks
### Future changes in WebDriver BiDi protocol
The WebDriver BiDi protocol may still undergo specification changes that could impact WPT tests.
#### Mitigation
* **Add commands on-demand.** To reduce the effects of future changes to the protocol, commands can be added as and when required instead of preemptively including all possible commands in the testdriver API.
* **Keep the testdriver API minimally required.** By keeping the testdriver API as minimal as possible, the impact of future protocol changes can be further reduced, as fewer API changes will be required.
### WebDriver BiDi implementation differences
Different implementations of WebDriver BiDi may have variations that could potentially complicate its use within the context of the WPT tests.
#### Mitigation
Ensure the testdriver API is compliant with the specification by avoiding reliance on implementation-specific data and behavior.
### WebDriver BiDi events might not agree with some observable side effects
In certain tests, the observable side effects may not correspond entirely with the WebDriver BiDi events. One such instance arises when a console.log BiDi event occurs, which does not guarantee the automatic display of the console message in the browser's developer tools.
#### Mitigation
This risk has a low priority impact and should not impede the project's progress.
### Different WebDriver BiDi implementation status
The implementation of WebDriver BiDi may vary across different endpoints, which can lead to tests utilizing BiDi functionality failing for endpoints lacking the required test components.
#### Mitigation
Expanding testdriver with WebDriver BiDi shouldn't cause previously passing tests to fail. This is because any regressions would only occur due to new tests that utilize WebDriver BiDi encountering incompatible endpoints - a scenario explicitly excluded from regression consideration.
### TODO: any other risks?
TODO
## Alternatives considered
The list of alternatives of different granularity levels:
### Flatten bidi and classic actions in `test_driver`
In this alternative, in [testdriver actions](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.1tdetff8j5en), the new WebDriver BiDi methods are added directly to the root testdriver object instead of introducing a dedicated `bidi` namespace. This approach has both advantages and disadvantages:
#### Advantages
No need in introducing a new namespace, and continue using the established pattern of adding new methods.
#### Disadvantages
* Potential for name clashes: If a new WebDriver BiDi action or event has the same name as an existing WebDriver Classic action or event, it could cause conflicts.
* Harder to determine if a specific testdriver method requires WebDriver BiDi support: Developers may need to consult the documentation or code to determine if a particular method requires WebDriver BiDi support.
### Make event emitting generic in testdriver
In this alternative, `test_driver` exposes an `EventTarget`, or methods `addEventListener(type, listener)`, and `removeEventListener(type, listener)`. Test will be able to subscribe to any WebDriver BiDi event.
#### Advantages
This approach is more flexible, as there will be no need in adding support for each WebDriver BiDi event.
#### Disadvantages
The API would depend on the WebDriver BiDi, and writing / reading tests would require being familiar with it.
### Do not expose `test_driver.bidi.{module}.{event}.subscribe()`, do it silently in the `on`method
In this alternative, the `test_driver.bidi.{module}.{event}.subscribe()` method is not exposed in the testdriver API. Instead, each `on` handler subscribes to the required event. The API becomes simpler, as test writers would not require care about the concept of WebDriver BiDi subscriptions at all. Nonetheless, this option was rejected because the `once` handler cannot be implemented since subscription is an asynchronous process.
### Expose generic  `session.subscribe` method in `test_driver` instead of `test_driver.bidi.{module}.{event}.subscribe()`
In this alternative, the `test_driver.session.subscribe` method is exposed in the testdriver API. This allows for more flexibility, as all the BiDi events are supported by default. This provides more flexibility because all BiDi events are natively supported. This suggestion was rejected because the API is challenging to use. Test writers must be familiar with WebDriver BiDi, and there is a risk of [future changes in WebDriver BiDi protocol](https://docs.google.com/document/d/1BSylQU7rtF_BAOK-muV8Ttle97rzjGgaeKKT6070m1k/edit?tab=t.0#bookmark=id.rq4z3shbztxb), which is a concern.
### Add async_execute_script to BaseProtocolPart
In this alternative, the `BaseProtocolPart` should be extended with the async `async_execute_script` method. The alternative is rejected, as it requires significant changes in the WebDriver Classic client.
### Add `AsyncProtocolPart` with `get_loop` method
Instead of exposing the event loop in the WebDriverBidiProtocol, add a dedicated `AsyncProtocolPart` protocol part, which exposes the loop via `get_loop` method. The alternative is rejected, as the event loop is not a part of the protocol, but rather the implementation detail.
### Extend `CallbackHandler` to `support async actions`
In this alternative, the async actions are supported by `CallbackHandler`. This would require making a `CallbackHandler` of the event loop in order to schedule actions processing. The alternative is declirejectedned, as it would be required to update all the `CallbackHandler` usages.
### Extend testdriver - testrunner transport with async_action
In this alternative, the testdriver - testrunner transport is extended with a special `async_action` type. Rejected, as testdriver should not be aware of the specific actions implementation. From the testdriver perspective all the actions are asynchronous.
