# RFC 183: Remove experimental `dump_test_results` function in <testharnessreport.js>

## Summary
That function is still called but the element it creates is never accessed. The created element contains stringified JSON of test result data.
PR [^1] proposes to remove the function. It was added in 2015 with the comment "This is experimental until we have a better way to integrate with saucelabs".
PR [^2] reduces the performance of the generating the test results' summary table which is consumed by `dump_test_results`.
If that function is never used, degraded performance doesn't matter.

## Details
See [^3] and [^4].

## Risks
None known as the element generated from the function seems unused in WebKit, Gecko and Chromium; see [^3].
It could be used in other downstream-projects, though.

[^1]: https://github.com/web-platform-tests/wpt/pull/45003
[^2]: https://github.com/web-platform-tests/wpt/pull/45002
[^3]: https://github.com/web-platform-tests/wpt/pull/45003#issue-2176117406
[^4]: https://github.com/web-platform-tests/wpt/issues/44352#issuecomment-1985862862
