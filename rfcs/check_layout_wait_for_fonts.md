# RFC 213: Wait for `document.fonts.ready` before running `checkLayout()` tests

## Summary

Change [`checkLayout()`][check-layout-th] to wait for
[`document.fonts.ready`][fonts-ready] to fulfill before running tests.

[check-layout-th]: https://github.com/web-platform-tests/wpt/blob/9952f6fa233ad9e067b2ff28ab32832cc6d8fd66/resources/check-layout-th.js#L211
[fonts-ready]: https://developer.mozilla.org/en-US/docs/Web/API/FontFaceSet/ready

## Details

### Motivation

https://github.com/web-platform-tests/rfcs/pull/22 removed the requirement that
[Ahem] is a system font.
Since then, tests must load Ahem as a web font before use.

testharness.js runs [synchronous `test()` functions][test] as soon as they're
registered, which can be before Ahem and other necessary web fonts are loaded.
To avoid testing against a layout with a default font in this scenario, [many
tests] wait for `document.fonts.ready` to fulfill first.
However, test authors can still forget to use this boilerplate ([example 1],
[example 2]), especially since a cached or system-installed font can mask the
underlying flakiness.

[Ahem]: https://web-platform-tests.org/writing-tests/ahem.html
[test]: https://web-platform-tests.org/writing-tests/testharness-api.html#test
[many tests]: https://github.com/search?q=repo%3Aweb-platform-tests%2Fwpt+%2Fdocument.fonts.ready.*%28checkLayout%7Ctest%29%2F&type=code
[example 1]: https://crrev.com/c/5943572
[example 2]: https://crrev.com/c/5889804

### Proposed Changes

* [`check-layout-th.js`][check-layout-th] is a helper script for generating
  layout-related `test()`s, which should generally run against a stable final
  layout.
  Therefore, in `check-layout-th.js`, change `setup(...)` to
  `promise_setup(() => document.fonts.ready, ...)` and `test(...)` to
  `promise_test(async ...)` accordingly.
  The `promise_test()`s [wait for `promise_setup()`][promise-setup], but will still run
  sequentially in the order in which they were registered.
* Clean up obsolete [`document.fonts.ready.then(() => checkLayout(...))`][many
  tests] and similar ad hoc patterns.
* To reflect WPT's minimum requirements, stop running tests in
  web-platform-tests/wpt Taskcluster/Azure with
  [`--install-fonts`][install-fonts].
  Without system fonts, stability checks will be more likely to block tests
  that don't wait for web fonts.

[promise-setup]: https://web-platform-tests.org/writing-tests/testharness-api.html#promise_setup
[install-fonts]: https://github.com/web-platform-tests/wpt/blob/38c69461a2178d1c9fa80a948e393612e0220c95/tools/wptrunner/wptrunner/wptcommandline.py#L273

### Alternatives Considered

Improving `checkLayout()` alone might not suffice, since [tests that use
`test()` directly][example 2] might still run with incorrect fonts.
However, delaying `test()` functions to run after `document.fonts.ready`
fulfills likely breaks [~100s of tests] that [assume the existing eager
scheduling behavior].

[~100s of tests]: https://chromium-review.googlesource.com/c/chromium/src/+/5980648?checksPatchset=2&tab=checks
[assume the existing eager scheduling behavior]: https://github.com/web-platform-tests/wpt/blob/f26694cede6d5602f01e7f7762876ec43cf74951/css/css-counter-styles/counter-style-at-rule/support/counter-style-testcommon.js#L20-L44

## Risks

https://crrev.com/c/5979634/1 tests a prototype of this RFC in Chromium CI:
* Fewer than 10 tests register additional `test()`s after `checkLayout()`,
  which is not allowed if `checkLayout()` registers `promise_test()`s instead.
  This is trivially fixed by applying the same `test(...)` to
  `promise_test(async ...)` transformation.
* Other regressions/flakiness due to brittle `checkLayout()` tests relying on
  the existing scheduling behavior seem unlikely.
