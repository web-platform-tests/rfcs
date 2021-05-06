# RFC #72: Address space overrides

## Summary

The goal of this RFC is to enable web platform tests to cover the
[Private Network Access](https://wicg.github.io/private-network-access)
specification entirely. More specifically, to enable tests to exercise user
agent behavior in the face of responses served from `local`, `private` and
`public`
[address spaces](https://wicg.github.io/private-network-access#ip-address-space).

This is achieved by passing configuration parameters to the browser under test
forcing it to artificially consider specific (IP address, port) pairs to be
`local`, `private` or `public`.

This RFC aims to fix
[web-platform-tests/wpt#26166](https://github.com/web-platform-tests/wpt/issues/26166).

## Details

### Browser changes

A new configuration surface is added to each browser (it need not be uniform,
though that would certainly help with implementation) that allows overriding the
address space derived from specific (IP address, port) endpoints. For example,
this could be a command-line flag:

```sh
--ip-address-space-overrides=127.0.0.1:8000=private,127.0.0.1:8001=public
```

Test-only code wires this command-line flag to the code in the browser under
test that determines the address space of a network endpoint.

### Test environment changes

New ports are introduced on which to serve artificially-`private` and
artificially-`public` resources. These are named after the existing protocols
with `-private` and `-public` suffixes. Concretely, the server `Config` object
sees its `ports` field extended with entries like the following:

```python
{
  "http-private": [8002],
  "http-public": [8003],
  "https-private": [8445],
  "https-public": [8446],
}
```

The `wptserve` and `wptrunner` infrastructure automatically starts new servers
on each of these ports serving the appropriate protocol.

### Test runner changes

The web platform test runner sets browser-specific configuration options such
that the browsers implement the custom address space mapping described above
during tests.

This allows web platform tests to exercise browser behavior in the face of
`private` and `public` IP addresses, simply by targeting the above ports.

For example, a JS test wishing to make a request to a `public` server can
make use of the server-side substitution feature of `wptserve` to do so:

```
fetch("http://{{host}}:{{ports[http-public][0]}}/foo.jpg")
```

## Risks

None identified. This approach is fed by implementation experience.

## Alternatives considered

### Override based on IP address, not port

This approach was initially submitted as PR #72, but implementation experience
pointed out its shortcomings - see issue #79. The main issue is that it assumes
the `127.0.0.0/8` IPv4 subnet is routed to loopback on all machines, but that is
not true on macOS and other BSD-derived operating systems.

Instead of overriding the IP address space based on both IP address and port,
this approach simply overrides the IP address space for certain IP addresses.
Given that `127.0.0.0/8` *should* be routed to loopback, this approach makes use
of other IP addresses than 127.0.0.1 to run `wptserve` on.

The following IP addresses see their IP address space overridden:

```
127.1.0.1: private
127.2.0.1: public
```

The configuration surface added to browsers allows overriding per address. It
is otherwise very similar to that described in the proposal above.

The WPT infrastructure is modified to run new servers on these IP addresses, by
the way of two new domains: `public-web-platform.test` and
`private-web-platform.test`. These are added to the system hosts file. All
subdomains and protocols are supported on these new domains.

See web-platform-tests/wpt#28768 for an abandoned experimental implementation.

### Override addresses themselves

This approach was suggested by @ddragana.

A similar configuration mechanism is exposed by browsers by which the test
runner can override some IP addresses, such that the browser under test will
consider that sockets connected to IP A are in fact connected to a fake IP B.

For example, as a command-line flag:

```
--test-ip-address-override=127.1.0.1:8.8.8.8,127.2.0.1:192.168.1.1
```

Would force the browser to consider sockets connected to `127.1.0.1` as instead
being connected to `8.8.8.8` and likewise for `127.2.0.1` to `192.168.1.1`.

This approach allows overriding the address space without introducing the notion
of address spaces to the configuration surface. @ddragana argues that it is less
failure-prone than the above proposal due to it being simpler to implement.

In Chromium, however, this approach likely would require plumbing the fake IP
address lower into the network stack than we would have to plumb a fake address
space. Thus it seems that instead this would be harder to implement correctly.

### Use individual IP addresses

Same as above, but allow overriding address spaces per IP address instead of
per subnet.

The marginal difficulty of overriding per subnet seems low, and provides plenty
of test IP addresses to exercise intra-address-space, cross-ip-address requests,
which might come in handy [in the
future](https://github.com/WICG/private-network-access/pull/1#issuecomment-721110250).

### Use real IP addresses

If we need to test user agent behavior in the face of a variety of IP addresses,
then... just do it!

This approach would have the web platform test runner configure additional
loopback network devices and assign them `private` and `public` IP addresses.
Alternatively, the test runner could probably reroute traffic heading for these
IP addresses to `127.0.0.1` through some iptables magic. Then we could configure
the hosts file to resolve specific domains to those IP addresses, and test the
whole shebang end to end.

This approach's main pro is that it truly is an end to end test of the feature,
exercising production code to its full extent. By virtue of requiring no
test-only hooks, it also requires no work on the part of browser developers.

Its main con is that it requires the test runner to have system privileges. One
could design around that by restricting system privileges to the test setup
phase. That refactoring however would require a fair amount of work to ensure no
setup step is broken because of the IP address hijacking.

Additionally, it requires work to support for each test execution environment.

Finally, it arguably requires the purchase of a public IP address for our use.
If not, then we open ourselves up to esoteric failures due to the hijacking of a
legitimate IP address for test purposes, likely in a long enough time for
everyone to have forgotten the existence of this hack.

### Extend WebDriver

This alternative was initially recommended by @stephenmcgruer in
[web-platform-tests/wpt#26166](https://github.com/web-platform-tests/wpt/issues/26166).

The gist of this alternative is this: extend WebDriver to allow overriding
the address space derived from specific IP addresses or domains.

Its main pro is that such an approach would be standardized at a higher level
than web platform tests alone.

I reached out to some WebDriver experts to explore this avenue first. I was
informed that WebDriver's audience is web developers seeking to test their
creations rather than browser developers. As such, they recommended a simpler
approach with command-line flags instead.
