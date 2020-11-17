# RFC 71: Support adding URI fragment for reftests

## wpt issue

https://github.com/web-platform-tests/wpt/issues/26364

## Summary

Adding URI query and fragment parts are allowed for testharness.js tests via a
`<meta name="variant">` in the test file. This is a request to add support for
the same meta element for reftests. Text fragment anchors are only allowed as
top level navigations and requires support for adding a fragment to the test
url.

## Details

A text fragment url for highlighting text in a page can be done with fragments
like "#:~:text=match". The browser is not allowed to navigate to such elements
setting location.hash or clicking `<a href="#:~:text=match">`. One way of
allowing it as part of the test is to add
`<meta name="variant=" content="#:~:text=match">`, but that is currently only
supported for javascript tests like testharness tests.

There is a pseudo-element `::target-text` which can be used to style the
highlighting of text fragments. Testing the visual output is typically done
via reftests, and supporting variants for reftests would make it possible to
visually test this pseudo element.

A patch that supports this in the Chromium port can be found here:

https://chromium-review.googlesource.com/c/chromium/src/+/2537679


## Risks

* There is a question what it means to have multiple variants for ref-tests.
  Should the same set of reference apply to all variants? Should the same
  variants be added to the ref-urls, etc.
