# RFC #67: Add WebSockets over HTTP/2 support to wptserve

## Summary

Enable wptserve to support WebSockets over an existing http2 connection.

## Details

Allow tests to open a WebSocket over an existing http2 connection.

wptserver needs to implement the [Bootstrapping WebSocket with HTTP/2 protocol](https://tools.ietf.org/html/rfc8441).

It means that it needs to be modified to:
- Set ENABLE_CONNECT_PROTOCOL settings to 1
- Detect Extended CONNECT Method and implement the WebSocket opening handshake.
- pywebsocket operates on Apache mod_python Request objects. So we need to wrap H2Response and H2Request into a Request-like object.

## Risks

WebSocket tests who source **websockets/constants.js** have a wss variant. We will need to add a new variant named **http2** in order to use the http2 port when opening the WebSockets. WPT server does not support insecure HTTP/2 so we don't need to expose an insecure HTTP/2 WebSocket variant.

This change needs to update hyper libraries to a more recent version.

No other risks are expected except those introduced by any feature e.g. complexity.
