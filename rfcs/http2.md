# RFC 38: Enable http/2 server

## Summary

Enable the http/2 server by default, configured to serve on port `9000`. Add a filename flag `.h2.` that enables loading top-level tests in http/2 mode.

## Details

There has been a HTTP/2 server implementation in the tree for some time, but it was previously disabled due to problems running the tests on Travis. Now that we are no longer using Travis, it makes sense to enable the server.

By default this server is configured to run on port 9000.

In addition, actually using the server for testing is easier if one can directly load top-level tests on the server. For the https server we do this with a filename flag `.https.`. By analogy, the HTTP/2 server will work the same way with a `.h2.` flag.

## Risks

Running an extra server is always some risk since it may cause instability.

Vendor CI systems may not allow the use of port `9000` for some reason.
