# RFC 112: Add a Proxy Automatic Config file (PAC) to test connection timing/hints

## Summary

Use a proxy to create artificial connection delays and use fresh domain names for tests.

## Details

The [Resource Timing](https://w3c.github.io/resource-timing/) spec contains information about
connection timing (time spent in DNS resolution, creating a connection, handshake etc).

So far it has been difficult/impossible to test this reliably in WPT, as connections are usually
pooled and it's difficult to assert that a connection made from a test to one of the predefined
WPT domains is fresh.

The same goes for [preconnect](https://html.spec.whatwg.org/#link-type-preconnect) - testing that
preconnecting has an effect would require creating artificial connection delays and check that a
preconnect eliminates those delays.

Suggesting to generate the delays using a [Proxy Automatic Config file](https://developer.mozilla.org/en-US/docs/Web/HTTP/Proxy_servers_and_tunneling/Proxy_Auto-Configuration_PAC_file) that would route arbitrary hosts to
the web platform tests server and would generate artificial resolution delays.

## Risks

* Relies on a non-standard feature (PAC). However, that feature is very old and widely supported.
* Testing timing information is sensitive to raciness, but that's the nature of the feature and
  not directly related to proxies.
