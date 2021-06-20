# RFC 85: WebTransport over HTTP/3 Test Server

## Summary

Add a [WebTransport over HTTP/3](https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http3-01) server to wpt. The server is used to test the [WebTransport Web APIs](https://w3c.github.io/webtransport/).

The server will use a new [file name flag](https://web-platform-tests.org/writing-tests/file-names.html) `.wt.h3` (`.wt.h3.py`) for handlers. Handlers are python scripts which are used to respond WebTransport events.

This RFC shares the same background/motivation with [RFC 42](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/quic.md) but focuses on WebTransport over HTTP/3. See RFC 42 for more background.

## Details

### Implementation

The WebTransport over HTTP/3 test server is built on top of `aioquic`, which is already added under `tools/third_party`. As of version 0.9.11, `aioquic` lacks some mechanisms to implement a WebTransport server (e.g. configuring SETTINGS HTTP/3 frame) but we plan to update `aioquic` once it supports necessary mechanisms.

The server will be implemented under `tools/webtransport` directory.

### Handlers

A WebTransport handler is a Python script which contains a function `handle_event()`. Here is an example handler which echos back received data.

```python
from webtransport import (
    WebTransportSession,
    BidirectionalStreamDataReceived,
    DatagramReceived,
    WebTransportEvent
)

def handle_event(session: WebTransportSession,
                 event: WebTransportEvent) -> None:
    if isinstance(event, DatagramReceived):
        session.send_datagram(data=event.data)
    elif isinstance(event, BidirectionalStreamDataReceived):
        session.send_stream_data(
            stream_id=event.stream_id, data=event.data)
```

`session` is an object that represents a WebTransport over HTTP/3 session. It provides APIs to handle the session. Current proposed APIs can be found [here](https://bashi.github.io/wpt-wt-test-server-api/session.html).

`event` represents a WebTransport event such as a reception of stream data. Current proposed events can be found [here](https://bashi.github.io/wpt-wt-test-server-api/events.html).

#### Alternative handler design

Alternatively, we can define a set of functions which are called upon each event. This API pattern is similar to existing [H2 handlers](https://web-platform-tests.org/writing-tests/h2tests.html). The equivalent of the above example might look like:

```python
def datagram_received(session: WebTransportSession,
                      data: bytes):
    session.send_datagram(data=data)


def bidirectional_stream_data_received(session: WebTransportSession,
                                       stream_id: int,
                                       data: bytes):
    session.send_stream_data(stream_id_id=stream_id, data=data)
```

A major downside of this approach is that keeping API documents and implementations in sync would require extra works, as H2 handler API document duplicates comments from the source.

Another downside of this approach is that we won't have editor/IDE supports on what kind of callback functions are available.

TODO: Spend more time on the API design. People may prefer this alternative approach.

### Handler lookup

When the server receives an [extended CONNECT method](https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http3-01#section-3.2) for WebTransport, it looks up a corresponding handler from the directory specified by a configuration. The configuration is passed by `wptserve`. If the server finds a corresponding handler the server creates a new WebTransport session with the handler and starts passing events to the handler.

The server uses `":path"` header field of an extended CONNECT request to look up a handler. For example, if `":path"` is `/echo`, the server looks up `echo.webtransport.py` from the handler directory.

### File name flags

A new file name flag `.wt` is used to indicate a Python script is a WebTransport handler.

`.h2` and `.h3` flags indicate that the underlying protocol (HTTP/2 or HTTP/3).

### `wptserve` integration

`wptserve` runs the WebTransport over HTTP/3 test server in a way similar to [pywebsocket](https://github.com/web-platform-tests/wpt/blob/246a32576020cb9c4241b7cfbc296f92d944ff6b/tools/serve/serve.py#L713): `wptserve` launches the server as a daemon and passes relevant configurations (port number, handler directory etc) to the server. The daemon is monitored by `wptserve` like other daemons.

### Dependencies

As of writing this RFC, the only dependency is `aioquic`.

### `tools/quic` deprecation

There are no ongoing standardlization efforts which require a general QUIC server. We might remove `tools/quic` directory at some point.

## Risks

Risks are similar to [RFC #42](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/quic.md#risks).

The server itself is standalone and the maintenance cost of the code will be low from the `wptserve`'s perspective. The Web Networking team and the Ecosystem Infra team at Google will be responsible for maintaining the integration point and the server.
