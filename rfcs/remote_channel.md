# RFC 98: Cross-Window Channels

## Summary

Provide a channel API which provides support for server-mediated
messaging between browsing contexts, including those which are in
different browsing context groups, and so cannot communicate purely
from the client.

## Details

### Use Case

The modern web platform has features which allow isolating different
browsing contexts and scripting environments from each other such that
they are unable to communicate. For example windows opened with the
`noopener` attribute will not have a handle to their parent, nor will
the parent have a handle to the child. Similarly, cross-origin
navigations with the `cross-origin-opener-policy` header appropriately
set create a new browsing context group which is totally isolated from
the prior context. Typically in implementations each browsing context
group is assigned a unique OS-level process.

This creates some problems for testing; because testharness tests are
running in a single browsing context, and tests that involves a context
that is isolated from the context containing the test will be unable
to communicate test results back to the harness using web platform
APIs.

Given the importance of isolation in the modern web platform, it's
important to improve support for tests involving multiple browsing
context groups in web-platform-tests. Otherwise test authors will
either not write tests for these scenarios, or write them using
browser-specific techniques.

### Prior Art

* Gecko tests are able to provide cross-context communication using the
  parent (i.e. browser UI) process as an intermediary. The
  [SpecialPowers.spawn](https://searchfox.org/mozilla-central/source/testing/specialpowers/content/SpecialPowersChild.jsm#1547-1584)
  API available to gecko tests allows running a function in another
  context, using structured clone to pass the arguments
  ([example](https://searchfox.org/mozilla-central/source/dom/tests/browser/browser_data_document_crossOriginIsolated.js#17)).

* The
  [cross-origin-opener-policy](https://github.com/web-platform-tests/wpt/tree/7b0ebaccc62b566a1965396e5be7bb2bc06f841f/html/cross-origin-opener-policy)
  tests currently in web-platform-tests use a bespoke
  [dispatcher.py](https://github.com/web-platform-tests/wpt/blob/7b0ebaccc62b566a1965396e5be7bb2bc06f841f/html/cross-origin-opener-policy/resources/dispatcher.py)
  to allow cross-origin communication. This uses a queue (implemented
  as a list) stored in the wpt server stash object as a message
  channel. Each context is given a UUID in a URL parameter named
  `uuid`, and uses HTTP GET requests to check for any message in the
  queue stored in the stash entry with that UUID. Other contexts that
  know the UUID can POST a message that's appended to the queue.

* WebDriver also has the ability to communicate between different
  contexts, and [RFC
  89](https://github.com/web-platform-tests/rfcs/pull/89) proposed
  adding an `execute_script` method to testdriver to enable running a
  script in a different context. However the design of WebDriver and
  wptrunner means that only one context can communicate using
  WebDriver at a time, and for testharness we must always return
  control to the test window, since WebDriver is also used to
  communicate test results back to the harness. The single threaded
  nature of WebDriver, and requirement that all testdriver actions go
  via the main test window, make it hard to build an ergonomic
  cross-context messaging API.

All the above examples of prior art have some attractive features, and
it's possible to combine them in a way that should provide an
ergonomic API for web-platform-tests

* The Gecko SpecialPowers API appears to offer the best ergonomics,
  providing a relatively high level API for executing script in a
  remote context. However obviously the implementation depends on
  gecko specifics.

* The dispatcher framework provides an addressing mechanism (the
  `uuid` parameter in URLs) that is workable in the context of
  web-platform-tests, and the use of the server stash as the backend
  makes sense for wpt. However the requirement to poll the server for
  messages, and the relatively low level API based on passing strings
  around, seem like areas for improvement.

### Proposal

The following subsections will set out a proposal for an API that
combines some of these strengths.

#### Addressing

Contexts that want to participate in messaging must have a query parameter
called `uuid` in their URL, with a value that's a UUID. This will be
used to identify the channel dedicated to messages sent to that
context. If the context is navigated the new document may reuse the
same UUID if it wants to share the same message queue. Otherwise it's
an error to use the same UUID for multiple contexts. Trying to create
more than one simultaneous reader channel for the same UUID will fail.

#### Backend

The message passing backend is based on a Python
[Queue](https://docs.python.org/3/library/queue.html) object per
channel, stored in the server stash. The advantage of using queues
here is that they are designed to allow blocking reads. This means we
don't have to use a polling API but can, on the server side, wait for
a message to be added to a channel, and immediately forward the
message to the client. Because the stash itself runs in a single
Python process and communicates with the server processes via IPC, it
is sufficient to use a thread-based `Queue` rather than requiring a
full process per queue.

#### Low-Level API

The low-level API for messaging is based on the concept of
Multiple-Producer-Single-Consumer (MPSC) channels. Each channel is
identified by a UUID. When a channel is created a pair of objects are
returned, a `RecvChannel` to read from the channel and a `SendChannel`
to write to the channel. The `SendChannel` may be cloned to create
new writers, but only a single `RecvChannel` per channel is
permitted. This limitation is deigned to enforce good design in which
there are no races between different readers to get a message. In
particular, the intended use is not for broadcasting tasks to multiple
consumers, but to message a specific browsing context. Browsing
contexts will be able to create a `RecvChannel` with the UUID in their
URL and use that to receive messages from other contexts.

`Channel` acts approximately like an event target; consumers can call
the `addEventListener(type, fn)` method to add a event callback
function. The function is called with an object `{type, data}`, where
`type` is the event type and `data` is type-specific data. All
channels support `connect` and `close` events. Callbacks can be
removed using `removeEventListener(type, fn)`.

A `SendChannel` has a `send(obj)` method. This causes the object to be
encoded, first using the remote object serialization (see below) and
then as a JSON string, before being sent to the queue.

A `RecvChannel` must call its async `connect()` method to start
receiving messages. Messages are first deserialized from JSON and then
undergo remote object serialization. They are provided as event
callbacks with a type of `message`. A message handler function can be
registered using `addEventListener("message", callback)`; the object
passed to the callback function has the message in its `data`
property.  Alternatively the async `nextMessage()` function returns a
promise which resolves to next message received.

The implementation of channels is based on websockets, with one
websocket per `Channel` object (i.e. a different websocket is used for
a `SendChannel` vs a `RecvChannel` even when they correspond to the
same underlying queue). On the backend the websocket handler function
for a `SendChannel` listens for incoming messages and writes them to
the corresponding `Queue`, whilst the websocket handler for a
`RecvChannel` essentially blocks on the `get()` method for the `Queue`
and writes and message it receives to the socket. In detail some
additional care is needed to ensure that the websocket handler
functions shutdown when the socket is closed.

#### Remote Object Serialization

In order to allow passing complex JavaScript values between contexts,
a serialization format loosely based on the [WebDriver
BiDi](https://w3c.github.io/webdriver-bidi/#type-common-RemoteValue)
proposal is used. An object is represented using a JSON object as
follows:
```js
{
  type: Name of the value type,
  value: A JSON-compatiable representation of the value. Nested values are themselves
         represented using this serialization.
  objectId: A unique id assigned to the object in case of circular references.
}
```

So, for example, an array `a` with content `[1, "foo", {"bar": null}, a]` is represented as:

```js
{
  "type": "array",
  "objectId": 0,
  "value": [
    {
      type: "number",
      value: 1
    },
    {
      "type": "string",
      "value": "foo"
    },
    {
      "type": "object",
      "value": {
        "bar": {
          "type": null
        }
      }
    },
    {
      "type": "array",
      "objectId": 0
    }
  ]
}
```

This supports the following types:

* All JS primitive types
* Builtin types: `Date`, `Regexp`, `Error`
* Functions
* Collections: `Array`, `Map`, `Set`, `Object`
* `SendChannel`. This enables an important pattern: to receive
  messages from a remote context, you can send it a `SendChannel`
  object to use for responses.
* `RemoteObject`. This enables sending an arbitrary object as a
  reference that can be resolved in the original realm. For example an
  `Element` named `elem` can be transferred as a
  `RemoteObject(elem)`. If this `RemoteObject` is later transferred
  back to the original realm it will be converted back to the original
  object.

Deserialization creates values equivalent to the original values in
the current realm e.g. a serialized array is reconstructed as an array
object in the realm where deserialization occurs, and similarly for
each element in the array. `RemoteObject` differs slightly; given a
serialized `RemoteObject` referencing some object `obj`, if `obj`
doesn't exist in the current realm a new `RemoteObject` is created
referencing `obj`. If `obj` does exist in that realm, it is returned
as the result of deserialization.

#### Higher Level API

The low-level API provides required primitives, but it's difficult to
use directly. To achieve the aim of making tests no harder to write
than SpecialPowers-based Gecko tests, there is also a higher-level API
initially providing two main capabilities: `postMessage` which sends a
message to a remote global, and `call` which executes a provided
function in the remote global with given arguments.


This API is provided by a `RemoteGlobal` object. The `RemoteGlobal`
object doesn't handle creating the browsing context (or other global
object), but given the UUID for the remote, creates a `SendChannel`
which is able to send messages to the remote. Alternatively the
`RemoteGlobal` may be created first and its `uuid` property used when
constructing the URL.

Inside the remote browsing context itself, the test author has to call
`global_channel()` in order to set up a `RecvChannel` with UUID given
by the `uuid` parameter in `location.href`. By default this is not
connected until the async `connect()` method is called. This allows
message handlers to be attached before processing any messages. For
convenience `await start_global_channel()` returns an already
connected `RecvChannel`.

The `RecvChannel` object offers an `addMessageHandler(callback)` API
to receive messages sent with the `postMessage` API on `RemoteGlobal`,
and async `nextMessage()` to wait for the next message.

##### Script Execution

`RemoteGlobal.call(fn, ...args)` serializes the function
`fn`, using `Function.prototype.toString()`. Each argument is
serialized using the remote value serialization algorithm. Along with
the function string and the arguments, a `SendChannel` is sent for the
command response (only one such channel is created per `RemoteGlobal`
for efficiency reasons), and a command id is sent to disambiguate
responses from different commands.

On the remote side the function is deserialized and executed. If
execution results in a `Promise` value, the result of that promise is
awaited. The final return value after the promise is resolved is sent
back and forms the async return value of the `call` call. If
the script throws, the thrown value is provided as the result, and
re-thrown in the originating context. In addition an
`exceptionDetails` field on the response provides the line/column
numbers of the original exception, where available.

#### Navigation and bfcache

For specific use cases around bfcache, it's important to be able to
ensure that no network connections (including websockets) remain open
at the time of navigation, otherwise the page will be excluded from
bfcache. In the current prototype this is handled as follows:

* A `disconnectReader` method on `SendChannel`. This causes a
  server-initiated disconnect of the corresponding `RecvChannel`
  websocket. This is pretty confusing! The idea is to allow a page to
  send a command that will initiate a navigation, then without knowing
  when the navigation is done, send further commands that will be
  processed when the `RecvChannel` reconnects. If the commands are
  sent before the navigation, but not processed, they can be buffered
  by the remote and then lost during navigation. An alternative here
  would be a more explicit protocol in which the remote has to send an
  explicit message to the test page indicating that it's done
  navigating and it's safe to send more commands. But the way the
  messaging works, it's hard for a random page that's loaded to
  initiate a connection to the top-level test context For example,
  consider a test page T with a channel pair allowing communication
  with remote window A. If A navigates to B, there's no simple
  mechanism for B to create a channel to T. One could get around this
  by e.g. putting the uuid of the existing `SendChannel` from A to T
  into the URL of B and constructing it from there, but that's quite
  fiddly and doesn't work in cases where the URL is immutable
  e.g. history navigation.

* A `closeAllChannelSockets()` method. This just closes all the
  open websockets associated with channels in the context in which
  it's called. Any channel then has to be reconnected to be used
  again. This doesn't feel especially elegant since it's violating
  layering, but it does mean you can be relatively confident that
  calling `closeAllChannelSockets()` right before navigating will
  leave you in a state with no open websocket connections (unless
  something happens to reopen one before the navigation starts).

One possible alternative to this would be to build a poll-based
`RecvChannel` specifically for bfcache use cases. This would only
receive new messages when a `poll()` method is called, so would put
the remote page fully in control of network connections. Similarly a
`poll()` based `SendChannel()` could ensure that a network connection
is only created when actually sending a message.

### API Summary

```ts
/**
* Create a new channel pair
*/
function channel(): [RecvChannel, SendChannel] {}

interface ChannelEvent {
  type: string,
  data: Object | undefined
}

/**
* Channel used to send messages
*/
class SendChannel() {
  uuid: string

  /**
   * Create a SendChannel from a given UUID
   */
  constructor(uuid: string) {}

  /**
   * Connect to the channel. Automatically called when sending the
   * first message
   */
  async connect() {}

  /**
   * Close the channel and underlying websocket connection
   */
  async close() {}

  /**
   * Add a event callback funtion. Supported message types are "connect", and "close".
   */
  addEventListener(type: string, fn: (event: ChannelEvent) => void) {}

  /**
   * Remove an event callback function
   */
  removeEventListener(type: string fn: (event: ChannelEvent) => void) {}

  /**
   * Send a message `msg`. The message object must be JSON-serializable.
   */
  async send() {msg: Object}

 /**
  * Disconnect the RecvChannel, if any, on the server side
  */
  async disconnectReader() {}
}

/**
 * Channel used to handle messages. Not directly constructable.
 */
class RecvChannel() {
  uuid: string

 /**
   * Connect to the channel and start handling messages.
   */
  async connect() {}

 /**
   * Close the channel and underlying websocket connection
   */
  async close() {}

  /**
   * Add a event callback funtion. Supported message types are "connect", "close", and "message".
   */
  addEventListener(type: string, fn: (event: ChannelEvent) => void) {}

  /**
   * Remove an event callback function
   */
  removeEventListener(type: string fn: (event: ChannelEvent) => void) {}

 /**
  * Wait for the next message and return it (after passing it to
  * existing handlers)
  */
  async nextMessage(): Promise<Object> {}
}

 /**
 * Create an unconnected channel defined by a `uuid` in
 * `location.href` for listening for RemoteGlobal messages.
 */
async global_channel(): RemoteGlobalCommandRecvChannel {}


/**
 * Start listening for RemoteGlobal messages on a channel defined by
 * a `uuid` in `location.href`
 */
async start_global_channel(): Promise<RemoteGlobalCommandRecvChannel> {}


/**
 * Handler for RemoteGlobal commands. This can't be constructed directly
 * but must be onbtained from `global_channel()` or `start_global_channel()`.
 */

class RemoteGlobalCommandRecvChannel {
  /**
   * Connect to the channel and start handling messages.
   */
  async connect() {}

 /**
   * Close the channel and underlying websocket connection
   */
  async close() {}

  /**
   * Add a handler for `postMessage` messages
   */
  addEventListener(fn: (msg: Object) => void) {}

  /**
   * Remove a handler for `postMessage` messages
   */
  removeEventListener(fn: (msg: Object) => void) {}

 /**
  * Wait for the next `postMessage` message and return it (after passing it to
  * existing handlers)
  */
  async nextMessage(): Promise<Object> {}
}

class RemoteGlobal {
  /**
   * Create a RemoteGlobal. The dest parameter is either a
   `SendChannel` object or the UUID for the channel. If ommitted a new
   UUID is generated.
   */
  constructor(dest?: SendChannel | string) {}

  /**
   * Connect to the channel. Automatically called when sending the
   * first message
   */
  async connect() {}

  /**
   * Close the channel and underlying websocket connections
   */
  async close() {}

  /**
   * Disconnect the RecvChannel running in the remote context, if any,
   * on the server side
   */
   async disconnectReader() {}

  /**
   * Post the object msg to the remote, using JSON serialization
   */
   postMessage(msg: Object) {}

  /**
   * Run the function `fn` in the remote context, passing arguments
   * `args`, and return the result after awaiting any returned
   * promise.
   *
   * Arguments and return values are serialized as RemoteObjects.
   */
   async call(fn: (args: ...any) => any, ...args: any): Promise<any> {}
}

/**
 * Representation of a non-primitive type passed through a channel
 */
class RemoteObject {
  type: string;
  objectId: string;

  /**
   * Create a RemoteObject containing a handle to reference obj
   */
  static from(obj): RemoteObject

  /**
   * Return the local object referenced by the objectId of this RemoteObject,
   * or null if there isn't a such an object in this realm.
   */
  toLocal(): Object?

  /**
   * Remove the objectId from the local cache. This means that future
   * calls to `toLocal` with the same objectId will always return
   * `null`.
   */
   delete()
}

/**
 * Close all websockets in the current global that are being used for channels.
 */
function closeAllChannelSockets() {}
```

### Resource Management

With the stash-based approach, it's important to clean up the stash
once no channels remain. Otherwise we end up leaking queues. The
general approach is to explicitly store a refcount as part of the
stash value. Each socket that connects increments the refcount, and
decrements it once the socket is closed. Access to the stash is
protected by a lock, so only one socket may touch the refcount at a
time. In practice the fact that there can only be one `RecvChannel`
per queue means that we can just count `SendChannel` sockets and
enforce an at-most-one rule for `RecvChannel`. The underlying queue is
deleted when no socket connections remain. This does mean that if we
were to e.g. put a message on a queue, then delete all the channels
before forwarding the message, and then use the UUID to create a new
`RecvChannel` the queue would be deleted and the message lost. One can
imagine this scenario happening when navigating a remote context which
has a `RecvChannel` tied to the UUID in the URL, if the `SendChannel`
for the context isn't kept alive. Another possibility is to only
delete the queue if it's empty, and register an explicit completion
callback in the test that deletes any queue with a UUID known to the
test context. Of course this doesn't help if a channel is created in a
non-test context (e.g. for messaging between multiple remotes).

## Example

test.html

```html
<!doctype html>
<title>call example</title>
<script src="/resources/testharness.js">
<script src="/resources/testharnessreport.js">
<script src="/resources/channel.js">

<script>
promise_test(async t => {
  let remote = new RemoteGlobal();
  window.open(`child.html?uuid=${remote.uuid}`, "_blank", "noopener");
  let result = await remote.call(id => {
    return document.getElementById(id).textContent;
  }, "test");
  assert_equals("result", "PASS");
});
</script>
```

child.html

```html
<p id="nottest">FAIL</p>
<p id="test">PASS</p>
<script>
start_window_channel();
</script>
```

## Implementation Requirements

The implementation only depends on wptserve and normal content js; it
doesn't depend on testdriver or any test-only APIs. Therefore
integration into any deployment not using wptrunner/testdriver should
be straightforward.

## Possible Future Additions

Note: Implementation of any suggestions in this section would happen
in the context of a further RFC. This section is only to sketch some
possibilities for further development of the API.

The primitives here could be integrated more completely with
testharness.js. For example we could use a `RemoteGlobal` as a source
of tests in `fetch_tests_from_window`. Alternatively, or in addition
to that, we could integrate asserts with script execution, so that a
remote context could include a minimal testharness.js and enable
something like:

```js
promise_test(t => {
  let r = new RemoteGlobal();
  window.open(`file.html?uuid=${r.uuid}`, "_blank", "noopener");
  await r.call(() => {
    assert_equals(window.opener, null)
  });
});
```

Currently this is possible if `file.html` is put in `resources` or
`support` directory so it can't be detected as a test. But it would
perhaps be better to have a minimal subset of testharness available
without providing the API surface that only makes sense in a test
window.

It may be possible in the future to replace the backend with a
WebDriver BiDi based message passing system that would avoid the need
for websockets. By sticking close to WebDriver BiDi proposed semantics
the transition may even be seamless.

testdriver integration is possible. For example we could add
`RemoteContext.testdriver.set_permission(params)` to execute a
`set_permission` command in the remote context (and similarly for the
remainder of the testdriver API). This would desugar to
`testdriver.set_permission(params, uuid)` and testdriver would be
updated to identify the target window from the `uuid` parameter in its
`location.href`.

## Risks

This has been largely covered elsewhere in the document, but several
risks stand out:

* This is a relatively large API addition which will require ongoing
  maintenance.

* The websocket based transport adds complexity that can be hard for
  test authors to understand, particularly in tests that are
  themselves affected by the presence of the websocket connection
  (e.g. bfcache). The semantics around queue deletion may also be
  tricky. Broken tests may be hard to debug.

* Extensive use of promises in the API means using async/await is very
  desirable for readable code in the implementation and in the
  tests. This may cause issues with older js implementations. However
  this has been available in most browsers since around 2017, so it
  seems unlikely to be a real problem.

* The queue based backend increases the impact of resource leaks vs a
  backend based on lists, since additional python threads are used in
  the queue implementation.

* Having to manually pass around UUIDs to identify channels makes it
  easy to author broken tests (e.g. trying to reuse the same UUID
  across multiple windows).

## References

[PR 29803](https://github.com/web-platform-tests/wpt/pull/29803)
contains a prototype implementation of this.

<!--  LocalWords:  UUID WebDriver wptrunner testharness APIs UI
 -->
<!--  LocalWords:  testdriver websockets bfcache
 -->
