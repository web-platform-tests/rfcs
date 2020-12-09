# RFC 74: Add testdriver.js support for WebDriver delete_all_cookies commands

## Summary

Add testdriver.js support for the [WebDriver `delete_all_cookies` command](https://w3c.github.io/webdriver/#delete-all-cookies).

## Details

There are many different helper methods in WPT for setting, getting, and deleting cookies(see [cookies/resources](https://github.com/web-platform-tests/wpt/tree/master/cookies/resources), or the cookie-related `Response` methods for some examples). Some of them work well,
and others are prone to race conditions, or [don't cover all use cases](https://github.com/web-platform-tests/wpt/blob/d1aaf685cfb02e0350701550afe4146d555f2461/cookies/resources/cookie.py#L13-L17).

Rather than re-write existing support code to fix these issues, this RFC aims to add support for the WebDriver `delete_all_cookies` command. An PR exists at https://github.com/web-platform-tests/wpt/pull/26723`, as it addresses a known issue of [flaky leftover state](https://github.com/web-platform-tests/wpt/issues/26707).

Future RFCs may propose adding the rest of the cookie commands, once [cookie serialization is defined by WebDriver](https://github.com/w3c/webdriver/issues/1562). That will allow writing cookie tests independent of HTTP or DOM cookie APIs (i.e., `Set-Cookie`, `document.cookie`, `CookieStore`).

## Risks

This increases the API surface of testdriver.js, so it's more code to maintain or reason about. In addition, the developer experience of running local cookie tests will be less convenient (requiring `wpt run` instead of just hitting refresh).