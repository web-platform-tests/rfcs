# RFC 100: Add testdriver.js support for window minimize/restore (available in WebDriver and Marionette)

## Summary

Add testdriver.js support `minimize` and `restore`, which wrap the for the [WebDriver `minimize` command](https://w3c.github.io/webdriver/#minimize-window) and [`setWindowRect`](https://w3c.github.io/webdriver/#set-window-rect).

## Details

The [Page Visibility](https://www.w3.org/TR/page-visibility-2/) spec adds web-exposed insight into
whether the browser is "in the background". This can have different meanings on different platforms.
However, in WebDriver one scenario of this is available through `minimize`.

By allowing the test to minimize/restore the window, we can test how pages using the exposed APIs
interact with that behavior.

## Risks

Tests that use this API might not give consistent results when running in headless mode.
This is due to the nature of the `page visibility` API and APIs that require the browser to be
in a particular state with regards to the OS in order to succeed.

Also, some platforms (mobile)? may not support `minimize`, in which case tests using this feature
should fail early with a precondition.