# RFC 169: WebRTC Test Server


## Summary

Add a [WebRTC](https://datatracker.ietf.org/doc/html/rfc8834) server to wpt. The server is used to test the [WebRTC Web APIs](https://w3c.github.io/webrtc/). The server requires Python 3.

The server uses a WebSocket connection to negotiate the WebRTC connection.
It terminates STUN, DTLS and SRTP and forwards unencrypted RTP or RTCP packets to the browser.
This allows the tests to receive and inspect any packets received.

Future versions may use WebTransport instead of WebSockets for signaling and transport of packets to the client and allow the client to send packets over the WebSocket to be forwarded
via WebRTC.

## Details

### Implementation

The WebRTC test server is built on top of `aiortc`, similar to how the WebTransport/H3 test server is based on `aioquic`.

The server will be implemented under `tools/webrtc` directory.

### `wptserve` integration

When the WebRTC test server runs as a part of `wptserve`, `$ROOT` will be the same as `$WPT_ROOT`.

`wptserve` runs the WebRTC test server in a way similar to [pywebsocket](https://github.com/web-platform-tests/wpt/blob/246a32576020cb9c4241b7cfbc296f92d944ff6b/tools/serve/serve.py#L713):
`wptserve` launches the server as a daemon and passes relevant configurations (port number, root directory etc) to the server. The daemon is monitored by `wptserve` like other daemons.

### Dependencies

As of writing this RFC, the only dependency is `aiortc`.

## Risks

Risks are similar to [RFC #42](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/quic.md#risks).

The WebRTC test server itself is standalone and the maintenance cost of the code will be low from the `wptserve`'s perspective.
The WebRTC team at Google and Microsoft will be responsible for maintaining the integration point and the server.

