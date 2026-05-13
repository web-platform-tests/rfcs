# RFC 85: WebSocket over HTTP/3 Test Server

## Summary

Add a [WebSocket over HTTP/3](https://datatracker.ietf.org/doc/html/rfc9220) server to wpt for [WebSocket Web API](https://websockets.spec.whatwg.org/) tests. The server requires Python 3 and reuses existing pywebsocket3 handlers where possible.

This follows the [WebTransport over HTTP/3 test server](https://github.com/bashi/rfcs/blob/08cd404ee62df4dd93b11175310d44fa98187c96/rfcs/webtransport_h3_test_server.md) model and extends the [WebSocket over HTTP/2](https://github.com/bashi/rfcs/blob/08cd404ee62df4dd93b11175310d44fa98187c96/rfcs/websockets_http2.md) approach to WebSocket over HTTP/3.

## Details

### Implementation

The server will live under WPT's `tools/websocket_h3` directory and use `aioquic` for QUIC and HTTP/3. It advertises `SETTINGS_ENABLE_CONNECT_PROTOCOL = 1` and handles RFC 9220 Extended CONNECT requests with `:protocol` set to `websocket`.

After a successful handshake, DATA frames on the CONNECT stream carry RFC 6455 WebSocket frames.

### Handshake

The server validates that:

* `:method` is `CONNECT`;
* `:protocol` is `websocket`;
* an authority value is present; and
* a handler exists for the requested `:path`.

Unlike HTTP/1.1 WebSocket, RFC 9220 does not use `Sec-WebSocket-Key` or `Sec-WebSocket-Accept`. On success, the server sends `:status` `200` and treats subsequent DATA frames as WebSocket frame bytes.

### Handlers and lookup

Existing WebSocket WPT tests use pywebsocket3 handler files under `websockets/handlers` for server-side behavior such as echoing messages, selecting subprotocols, setting cookies, or closing connections.

The HTTP/3 server should reuse those handlers without a new handler format. It maps the Extended CONNECT `:path` to the same handler root, referred to here as `$WS_ROOT`. For example, `:path` `/echo` resolves to the existing echo handler under `$WS_ROOT`.

An adapter layer makes the HTTP/3 stream look like pywebsocket3 request and stream objects, so handlers can continue using the pywebsocket3 stream APIs.

### Static resources

The server also serves WPT static resources over HTTP/3 so the HTML test page loads on a QUIC session before the WebSocket is created. The subsequent WebSocket Extended CONNECT can then reuse that same origin's QUIC session.

Static serving reuses `wptserve` routing, handlers, `.sub.` template substitution, and configuration. The transport layer is new: H3 adapters convert HTTP/3 request details into `wptserve` style requests and write responses through `aioquic`.

### Test variants

Tests opt into HTTP/3 with the existing WPT flag mechanism:

```
// META: variant=?wpt_flags=h3
```

Helpers can then select `{{ports[h3][0]}}` and use `wss://` for the WebSocket URL. The HTTP/3 server remains opt-in so WPT does not start an extra QUIC server for unrelated runs.

### Integration

`wptrunner` should recognize `h3` tests but keep them disabled unless H3 support is explicitly enabled, for example with `--enable-h3`. When enabled, `wptrunner` includes those tests, and `wptserve` launches and monitors the WebSocket over HTTP/3 server.

Chromium should use the configured H3 port and force QUIC for that origin.

### Dependencies

The server depends on `aioquic`, `pywebsocket3`, and existing `wptserve` infrastructure. No new dependency is required.

### Out of scope

This RFC does not change the WebSocket API, add JavaScript API surface, or replace existing WebSocket servers. It only adds infrastructure to run existing WebSocket API tests over HTTP/3.

Support for WebSocket over HTTP/1.1 and WebSocket over HTTP/2 remains unchanged.

## Risks

The main risks are the extra opt-in daemon, the adapter between asynchronous HTTP/3 streams and pywebsocket3's synchronous handler API, and keeping static-resource behavior aligned with `wptserve`.

The Web Networking and Ecosystem Infra owners should maintain the integration point and server behavior.
