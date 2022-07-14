# RFC 118: Use a "proxy all" directive instead of PAC

## Summary

Proxy all traffic to the WPT server for certain test, when they need a fresh registrable domain.

## Details

See [RFC 112](https://github.com/web-platform-tests/rfcs/pull/112) for original reasoning.
The issue with using PAC files as previously suggested, is that they don't work on Mac. Mac only
supports setting the PAC URL in the OS settings, and not programatically. This is true not only for
Safari.

Most of the tests that require this need a fresh domain, but do not necessarily require the whole set
of features offered by PAC files.

So the proposal is that instead of relying on PAC as implemented and previously proposed, have a simpler
meta tag that allows proxying everything to the WPT server , e.g.
`<meta name="proxy" content="all">`.

## Risks

* As is with pac, more a limitation than a risk - by generating ad-hoc registrable domains we won't be able to use the certificate, which would limit that kind of test to   non-secure only.
