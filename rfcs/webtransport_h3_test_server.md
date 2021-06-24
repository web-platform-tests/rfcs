# RFC 85: WebTransport over HTTP/3 Test Server

## Summary

Add a [WebTransport over HTTP/3](https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http3-01) server to wpt. The server is used to test the [WebTransport Web APIs](https://w3c.github.io/webtransport/). The server requires Python 3.

The server uses handlers to handle WebTransport events. Handlers are python scripts which contains callback functions to respond WebTransport events.

This RFC shares the same background/motivation with [RFC 42](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/quic.md) but focuses on WebTransport over HTTP/3. See RFC 42 for more background.

## Details

### Implementation

The WebTransport over HTTP/3 test server is built on top of `aioquic`, which is already added under `tools/third_party`. As of version 0.9.11, `aioquic` lacks some mechanisms to implement a WebTransport server (e.g. configuring SETTINGS HTTP/3 frame) but we plan to update `aioquic` once it supports necessary mechanisms.

The server will be implemented under `tools/webtransport` directory.

### Handlers

A WebTransport handler is a Python script which contains some callback functions. Callback functions are called every time a WebTransport event happens. The current proposed callback functions can be found [here](https://bashi.github.io/wpt-wt-test-server-api/handler.html). The follwoing is an example handler which echos back received data.

```python
from webtransport import WebTransportSession

def stream_data_received(session: WebTransportSession,
                         stream_id: int,
                         data: bytes) -> None:
    session.send_stream_data(stream_id=stream_id, data=data)


def datagram_received(session: WebTransportSession,
                      data: bytes) -> None:
    session.send_datagram(data=data)
```

`session` is an object that represents a WebTransport over HTTP/3 session. It provides APIs to handle the session. Current proposed APIs can be found [here](https://bashi.github.io/wpt-wt-test-server-api/session.html).

`session` provides methods to create unidirectional and bidirectional streams. These methods are similar to `aioquic`'s [asyncio API](https://aioquic.readthedocs.io/en/latest/asyncio.html#aioquic.asyncio.QuicConnectionProtocol.create_stream). The following handler creates a bidirectional stream and writes "PASS" to the stream when a session is established.

```python
import asyncio
from webtransport import WebTransportSession

def session_established(session: WebTransportSession, path: str) -> None:
    loop = asyncio.get_event_loop()
    loop.create_task(notify_pass(session))


async def notify_pass(session: WebTransportSession) -> None:
    _reader, writer = await session.create_bidirectional_stream()
    writer.write(b"PASS")
    writer.write_eof()
```

### Handler lookup

When the server receives an [extended CONNECT method](https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http3-01#section-3.2) for WebTransport, it looks up a corresponding handler from the root directory specified by a configuration. In this RFC the root directory is referred to as `$ROOT`.

The server uses `":path"` header field of an extended CONNECT request to look up a handler. For example, if `":path"` is `/webtransport/handlers/echo.py`, the server looks up `$ROOT/webtransport/handlers/echo.py`.

This lookup mechanism provides flexibility to put handlers under multiple directories, but we will draw up a guideline to put handlers in a single directory for discoverability.

If the server finds a corresponding handler the server creates a new WebTransport session with the handler and starts passing events to the handler.

### File name flags

This RFC focues on WebTranport over HTTP/3. WebTransport over HTTP/2 is out of scope. However, in the future, we may consider adding `.h2` and `.h3` name flags to handler names to specify the underlying protocol (HTTP/2 or HTTP/3). A separate RFC will be submitted as needed.

### `wptserve` integration

When the WebTransport over HTTP/3 test server runs as a part of `wptserve`, `$ROOT` will be the same as `$WPT_ROOT`.

`wptserve` runs the WebTransport over HTTP/3 test server in a way similar to [pywebsocket](https://github.com/web-platform-tests/wpt/blob/246a32576020cb9c4241b7cfbc296f92d944ff6b/tools/serve/serve.py#L713): `wptserve` launches the server as a daemon and passes relevant configurations (port number, root directory etc) to the server. The daemon is monitored by `wptserve` like other daemons.

### Dependencies

As of writing this RFC, the only dependency is `aioquic`.

### `tools/quic` deprecation

There are no ongoing standardlization efforts which require a general QUIC server. We plan to remove `tools/quic` directory once the WebTransport over HTTP/3 test server is introduced to wpt.

## Risks

Risks are similar to [RFC #42](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/quic.md#risks).

The WebTransport over HTTP/3 test server itself is standalone and the maintenance cost of the code will be low from the `wptserve`'s perspective. The Web Networking team and the Ecosystem Infra team at Google will be responsible for maintaining the integration point and the server.
