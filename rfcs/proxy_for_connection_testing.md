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

Other use cases are listed [here](web-platform-tests/wpt#13465). In essence, specs that rely on
caching based on [network partition keys](https://fetch.spec.whatwg.org/#network-partition-keys)
can be tested more reliably if an arbitrary fresh [registrable domain](https://url.spec.whatwg.org/#host-registrable-domain) is created on the fly and can simulate a situation where a cold network partition is accessed. Note that tests that require ad-hoc registrable domains will be non-secure, as the certificate cannot be generated on the fly.

In addition, this can help test behavior on default ports, as PAC files can reroute fetches to
given ports.

Suggesting to generate the delays using a [Proxy Automatic Config file](https://developer.mozilla.org/en-US/docs/Web/HTTP/Proxy_servers_and_tunneling/Proxy_Auto-Configuration_PAC_file) that would route arbitrary hosts to the web platform tests server and would generate artificial resolution delays.

The URL for the PAC file would be declared in the testheader , e.g.
`<meta name="pac" content="resources/delay.js">`, which in some cases would mean that tests with a PAC
file would have to run in a separate browser session.


## Risks

* Relies on a non-standard feature (PAC). However, that feature is very old and widely supported.
* Introducing delays inside PAC can theoretically cause browsers to ignore it, need to pay attention to this.
* Testing timing information is sensitive to raciness, but that's the nature of the feature and
  not directly related to proxies.
* More a limitation than a risk - by generating ad-hoc registrable domains we won't be able to use the certificate, which would limit that kind of test to   non-secure only.
