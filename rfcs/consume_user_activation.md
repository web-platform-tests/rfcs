# RFC 128: Consume user activation 

Editor: Marcos Caceres, Apple Inc.

## Summary 

Tests in WPT are currently relying on various indirect means to "[consume user activation](https://html.spec.whatwg.org/#consume-user-activation)" [HTML].
Some of these means are non-standard in as far as they are not specified to consume user activation, meaning that the tests are inshrining non-standard behavior for conformance purposes.

The workarounds being used are as follows, with several drawbacks:

* Using Fullscreen API: non-standard, takes significant time to put an element into fullscreen. 
* Using window.open() and window.close() - non-standard, not mobile friendly. 
* Using Payment Request API - not widely implemented (e.g., not exposed in Gecko). 

We also looked at a range of other possible APIs that consume user activation, and found [none to be suitable](https://github.com/web-platform-tests/wpt/issues/36727#issuecomment-1296349964). 

## Details 

We would like to propose the addition of an async function `test_driver.consume_user_activation()` method that returns a `Promse<boolean>` (representing if the activation was consumed or not). For example: 

```
const consume = await test_driver.consume_user_activation();
```

The `consume_user_activation()` method can take a `Window` <var>context</var> (which defaults to the current Window object). Allowing consumption to happen at a particular window, if required. 

The `consume_user_activation()` method would be implemented via the proposed addition of “[Consume user activation of Window](https://github.com/w3c/webdriver/pull/1695)” to the Web Driver specification.

This prposal has several advantages:
 * it's fast - no opening windows or waiting for fullscreen to enter/exit.  
 * it's lightweight - it doesn't create new browsing contexts.
 * it's mobile friendly - as above.  
 * it's build for purpose - no more using other APIs to indirectly achieve the desired outcome.
 * It's really simple - it compliments `.bless()` and other user activaton functionality already available.  

[An implementation](https://github.com/WebKit/WebKit/pull/6539) is available in WebKit.  

## Risks

None known. 
