# RFC X: Add Custom Handlers support to testdriver.js

## Summary

Add testdriver.js support for [Custom Handlers](https://html.spec.whatwg.org/multipage/system-state.html#custom-handlers/), by
supporting a `setRPHRegistrationMode` command ([spec
PR])(https://github.com/whatwg/html/pull/8267).

## Details

The proposed `setRPHRegistrationMode` command places the user agent in a mode
where it will automatically accept or reject (depending on the mode) future
Custom Protocol Handlers registrations. This allows full end-to-end
testing of Custom Handlers API (eg, registerProtocolHandler), which normally [requires user
interaction](https://html.spec.whatwg.org/multipage/system-state.html#dom-navigator-registerprotocolhandler)
to confirm the user's consent to the protocol registration.

Links:
  - [WIP spec PR](https://github.com/whatwg/html/pull/8267)
  - [WIP web-platform-tests implementation PR](https://github.com/web-platform-tests/wpt/pull/35792)
  - [WIP Chromedriver implementation](https://chromium-review.googlesource.com/c/chromium/src/+/3865270)

### Alternatives Considered

I've also considered the usag of the already defined [Set Permission](https://w3c.github.io/permissions/#set-permission-command)
extension command, implemented at least by Chromedriver. In order to use such command, we would need to define a
new permission descriptor for the Custom Handlers feature. Such [proposal](https://github.com/whatwg/html/issues/7920)
has been discussed and rejected by Chrome and Firefox implementors.

## Risks

This increases the API surface of testdriver.js, so it's more code to maintain
or reason about.
