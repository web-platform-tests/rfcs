# RFC 74: Add testdriver.js support for WebDriver cookie commands

## Summary

Add testdriver.js support for the [WebDriver cookie commands](https://w3c.github.io/webdriver/#cookies).

## Details

There are many different methods currently for setting, getting, or deleting cookies across WPT (see [cookies/resources](https://github.com/web-platform-tests/wpt/tree/master/cookies/resources), or the cookie-related `Response` methods for some examples). Some of them work well,
and others are prone to race conditions, or [don't cover all use cases](https://github.com/web-platform-tests/wpt/blob/d1aaf685cfb02e0350701550afe4146d555f2461/cookies/resources/cookie.py#L13-L17).

Rather than re-write existing support code, this RFC aims to add support for the WebDriver cookie commands. An initial PR exists at https://github.com/web-platform-tests/wpt/pull/26723 to add support for `delete_all_cookies()`, as it addresses a known issue of [flaky leftover state](https://github.com/web-platform-tests/wpt/issues/26707).

This will also allow writing cookie tests independent of HTTP or DOM cookie APIs (i.e., `Set-Cookie`, `document.cookie`, `CookieStore`).

## Risks

This increases the API surface of testdriver.js, so it's more code to maintain or reason about. In addition, the developer experience of running local cookie tests will be less convenient (requiring `wpt run` instead of just hitting refresh).