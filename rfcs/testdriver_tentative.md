# RFC: Tentative automation APIs with testdriver-tentative.js

## Summary

Introduce a `testdriver-tentative.js` file alongside the current `testdriver.js`, for test automation APIs that are not backed by a published specification. Create a lint that ensures that any test that uses `testdriver-tentative.js` is itself tentative.

## Details

The Interop 2023 investigation effort on accessibility explored the option of a WebDriver (classic or BiDi) to expose the accessibility tree. A proof of concept `test_driver.get_accessibility_tree()` was implemented in https://github.com/web-platform-tests/wpt/pull/43773, but we currently require that everything in `testdriver.js` is backed by a spec'd WebDriver endpoint.

Allowing tentative automation APIs puts automation on the same footing as regular web-exposed APIs, where tentative tests have long been possible.

A sketch of how this would look like if implemented is here:
https://github.com/foolip/wpt/commit/a028f07d5e14f6c96426c0a9ffeb423a5c1a9ad3

The lint rule should be that any test that includes `/resources/testdriver-tentative.js` must also have "tentative" somewhere in the test path.

## Risks

As with other tentative tests, there is a risk of them never becoming fully specified, implemented across browsers and then made non-tentative. The main mitigation for this risk is the `is:tentative` search feature on wpt.fyi, used to exclude such tests. This was done during Interop 2024 planning and worked well.
