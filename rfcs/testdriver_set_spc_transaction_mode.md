# RFC 101: Add Secure Payment Confirmation support to testdriver.js

## Summary

Add testdriver.js support for [Secure Payment
Confirmation](https://w3c.github.io/secure-payment-confirmation/), by
supporting a `setSPCTransactionMode` command ([spec
PR](https://github.com/w3c/secure-payment-confirmation/pull/151)).

## Details

The proposed `setSPCTransactionMode` command places the user agent in a mode
where it will automatically accept or reject (depending on the mode) future
Secure Payment Confirmation authentications. This allows full end-to-end
testing of Secure Payment Confirmation, which normally [requires user
interaction](https://w3c.github.io/secure-payment-confirmation/#sctn-transaction-confirmation-ux)
to confirm the user's consent to the transaction authentication.

Links:
  - [WIP spec PR](https://github.com/w3c/secure-payment-confirmation/pull/151)
  - [WIP web-platform-tests implementation PR](https://github.com/web-platform-tests/wpt/pull/31345)
  - [WIP Chromedriver implementation](https://chromium-review.googlesource.com/c/chromium/src/+/3237267)

### Alternatives Considered

We also prototyped an approach where the WebDriver API was to accept/reject
just the currently-open SPC dialog (e.g. `webdriver.accept_spc_prompt`). This
design is cleaner in terms of being idempotent, however it [led to
flake](https://chromium-review.googlesource.com/c/chromium/src/+/3225868/11#message-443faa84dfec42cf930bf7bf99e47d35f784bbbc)
as the web-page is not able to tell exactly when the dialog has been displayed
to the user. For example, if the card art icon is slow to load, the test
web-page may race ahead of the UI being displayed and try to accept/reject it
before it even shows.

One could imagine defining more complex semantics there ('wait a reasonable
amount of time for an SPC dialog to exist, and then accept/reject it'), but
given that complexity the 'mode' approach seemed easier.

## Risks

This increases the API surface of testdriver.js, so it's more code to maintain
or reason about.
