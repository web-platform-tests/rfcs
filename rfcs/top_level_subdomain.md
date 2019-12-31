# RFC XX: Loading top-level tests on a subdomain.

## Summary

Add a `.www` [file name flag](https://web-platform-tests.org/writing-tests/file-names.html) that causes a given test to be loaded from the subdomain `www.web-platform.test` rather than the apex domain `web-platform.test`.

## Details

Tests are typically loaded from `http://web-platform.test/`. For some features that verify behavior within a site, it would be helpful to load the top-level document from a subdomain (e.g. `http://www.web-platform.test/`). Tests for cookies' `Domain` attribute are a good example: it's possible to write tests that verify that attribute's effects on requests from `www.web-platform.test` to `web-platform.test`, but requires creating a proxy document in a new window or frame, running tests in that context, and `postMessage()`ing results back to the main document. This adds a good deal of complexity that could be avoided if we opened the top-level document on the subdomain to begin with.

We distinguish files that ought to be loaded from `https://web-platform.test/` with an `.https` [file name flag](https://web-platform-tests.org/writing-tests/file-names.html).  We could support loading tests from `www.web-platform.test` by adding a `.www` flag.

## Risks

*   At some point, we might end up with a more-than-reasonable number of flags to consider when writing a test: `amazing-cookie-test.https.www.sub.tentative.any.js` might be a bit much...)
*   Naming is hard. `.subdomain` is appealing, but confusable with `.sub`.
