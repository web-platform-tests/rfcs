# RFC 108: Add testdriver.js support for WebDriver "Get All Cookies" and "Get Named Cookie" commands

## Summary

Add testdriver.js support for WebDriver commands that can be used to get cookies from the browser, ["Get All Cookies"](https://w3c.github.io/webdriver/#get-all-cookies) and ["Get Named Cookie"](https://w3c.github.io/webdriver/#get-named-cookie).

## Details

Some tests that assert how a browser treats certain cookies require a greater
level of introspection into the internal browser storage than is afforded to
websites. One example of this are tests that need to know whether a cookie was
saved with "Session" expiry or whether a certain expiry value was successfully
parsed. Writing those tests in WPT is sometimes impossible.

In other cases, writing an assertion to check the value of a cookie attribute
may not be impossible, but requires a complex setup of echo servers and
third-party requests that could be frustrating to test authors.

It would be great if WPT had the ability to call
`test_driver.get_cookies()` instead.

This has a dependence on agreeing to the right strategy for [cookie serialization
in WebDriver](https://github.com/w3c/webdriver/issues/1562), but there seems to
be agreement on the issue that preserving raw bytes without intermediate UTF-8
parsing seems like the right way to go.

## Risks

As mentioned, cookie serialization isn't fully defined (tough loosely agreed
upon) yet. However, an initial attempt at implementation might actually
significantly help inform and speed up the serialization question, so I think
this is a good thing.

Finally, these risks that were already surfaced in
[RFC 74](https://github.com/web-platform-tests/rfcs/pull/74) apply in this case
as well:
This increases the API surface of testdriver.js, so it's more code to maintain or reason about.
In addition, the developer experience of running local cookie tests will be less convenient (requiring `wpt run`).


