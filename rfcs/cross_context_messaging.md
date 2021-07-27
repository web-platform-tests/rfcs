# RFC 90: `testdriver` Add a cross-context messaging primitive

## Summary

Add an `send`/`poll`/`recv` methods to testdriver that can be used to
send messages to other browsing contexts, and receive messages for
the current browsing context.

## Details

In tests that involve multiple origins, or multiple browsing context
groups, there's often a need to make asserts about the behaviour of
browsing contexts other than the test context. This problem has been
solved by the COOP tests with
[dispatcher.js](https://github.com/web-platform-tests/wpt/blob/7b0ebaccc62b566a1965396e5be7bb2bc06f841f/html/cross-origin-opener-policy/resources/dispatcher.js). This
provides functions `send(uuid, message)` and `receive(uuid,
maybe_timeout)` which are used to send and recieve messages between
contexts given a shared key `uuid`. Typically the way this works is a
specific resource load is given a `uuid` via a query parameter in the
url. That resource will then `recieve()` messages using that `uuid`
whereas other contexts, such as that containing the main test, can
send messages using `uuid`.

The backend transport layer is the wptserve stash, which stores a list
of messages per `uuid`. A Python script `pop()`s the message from the
from of the queue when it handles a `GET` request with `uuid` as a
query parameter, and pushes the request body to the stash when it
handles a `POST` request, also with a `uuid` parameter.

This functionality is useful enough that it was adopted for Chrome's
bfcache tests, which suggests it's likely to be useful as a primitive
in many cases where tests involve multiple contexts or navigations.

It is proposed that we lift this functionality into testdriver itself,
integrating it with the context system used to target. The underlying
communication is poll-based, so as a generalisation we expose three
functions on `test_driver`, each of which returns a promise that is
resolved when the relevant action is complete:

```
test_driver.send(message, context);
test_driver.poll();
test_driver.recv(timeout=null)
```

`test_driver.send(mesage, context)` Sends `message` to the context
`context`. `message` must be JSON-encodable. Unlike the existing
implementation we don't send `message` directly, but instead wrap it
in a js object like:

```
{
  src: <uuid of sending context>,
  data: message,
}
```
Including `src` in the wrapper means we're always able to reply to a
message without needing to explictly communicate the uuid for the
source context.

`test_driver.poll()` polls the server for a message with the `uuid`
for the current context. The return value is `null` if there is no
message, or an object representing the wrapped message, if there is.

`test_driver.recv(timeout=null)` waits at most `timeout` ms to receive
a message, and throws an error if no message is received. If the
timeout is null, the function polls indefinitely.

As with the existing implementation, the backend is based on the
server stash. Since this is standard wpt infrastructure not tied to
wptrunner, it's possible to provide an implementation directly in
`testdriver.js` as part of `test_driver_internal`, meaning that it can
be overridden with a different transport if required, but there is no
downstream work required for implementors as part of this feature.

The use of the stash does present some promises; to avoid races if
multiple processes try to access the same stash item we need to take a
lock. This implies a performance cost to using the feature since all
requests are serialised, even if there's multiprocessing. This could
be reduced by using a per-key lock rather than a single global. The
existing code also has a limit to the number of concurrent requests on
the js side to avoid overwhelming the Python server.

## Example

```
let uuid = "{{uuid()}}";
window.open(`child.html?uuid=${uuid}`);
await test_driver.send("start_test", uuid);
let result = await test_driver.recv(uuid);
assert_equals(result, "PASS");
```

## Risks

The model in which we send messages to other contexts to get them to
do work leaves authors with a lot of challenges for e.g. error
handling. If the remote end throws an unexpected exception and fails
to reply as expected it's likely we end up with a hard-to-debug
timeout. It would be beneficial to supply some higher-level
functionality for running parts of tests outside the main test window
and communicating the result back to the text context.

The promise-based polling model means that the default style of
communication is command/response, where we `send()` a message to a
remote context and `await recv()` a result. To implement something like
events from the remote it would be necessary to build a custom event
loop that continually polls and updates local state as
necessary. Although this is possible, it would require repeated
polling of the server, which is inefficient. This could be addressed
by using Server-Sent-Events or websockets as the transport
mechanism. These also expose incoming messages as DOM events rather
than as promises. This probably makes simple cases more complex, but
is easier in cases where there are an unknown number of events.

## References

[RFC 86](https://github.com/web-platform-tests/rfcs/pull/86) contains
some discussion of the existing implementation and its reuse for bfcache.

[PR 29803](https://github.com/web-platform-tests/wpt/pull/29803)
contains a prototype implementation of this.
