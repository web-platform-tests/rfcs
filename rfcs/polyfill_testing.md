# RFC 111: Polyfill Testing

## Summary

Add a mechanism for loading one or more polyfill implementations of a web
platform feature and testing their correctness on existing web platform tests.

## Details

Polyfills are typically best effort implementations of browser features. It's
not uncommon for them to be outdated, broken or just not to implement all parts
of the specification they are polyfilling. Additionally, polyfills may only work
with certain browsers or with certain libraries. This proposal allows testing
polyfills using the same test suite we test browser compliance with. Being able
to know which tests the polyfill passes or doesn't pass is useful information
when developers are determining whether they can rely on it, just as they would
use this information to know whether they could rely on the feature in the
browser.

There are a few methods by which we could support this:

1. Rewrite the response from wptserve to insert a script tag which loads the
   polyfill before running tests.
2. Add the polyfill into testharnessreport.js.
3. Add the polyfill into a new /resources/polyfill.js which could also be
   included by reftests.
4. Using the [bootstrap scripts
   feature](https://github.com/w3c/webdriver-bidi/issues/65) once it is
   available.

## Risks

Depending on the method used there are various risks.

### Rewriting the response

Rewriting the response is a fairly minimal change to the infrastructure, and we
can always switch to another mechanism later transparently. The main downside is
that the test files loaded by the browser when a polyfill is injected are not
identical to the test files in the repo.

### Adding a canonical resource which includes the polyfill

Using the approach of having a specific resource to include will result in a
mixture of tests which support running with polyfills and tests which don't.
If we later adopt a mechanism which doesn't require explicitly including them
these artifacts may take some time to remove.

### Bootstrap scripts feature

This is most likely the best approach, however it is not yet available. The main
downside of this approach over one of the above approaches is that developers
cannot load the test URL directly in their browser to debug failures.
