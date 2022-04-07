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

There are a few methods by which we could support this but each of them
ultimately is some mechanism for running the polyfill script before running
tests.

## Risks

Depending on the method used there are various risks around how easily existing
tests can be run and observable side effects of having the polyfill injected
that could affect the test outcome.

### Rewriting the response

Rewriting the response is a fairly minimal change to the infrastructure. The
wpt server can insert the polyfill script into the files it serves before the
test content.

Advantages:
* Polyfill is transparently inserted across all existing tests (including ref
  tests).
* This is the same way that a polyfill would likely be loaded in a real
  scenario.
* As no modification is required, can easily switch mechanisms later.

Concerns:
* An extra script tag would affect tests which assume a static page structure,
  e.g. `document.getElementsByTagName('script')[2]` would no longer refer to the
  same script. We can mitigate this concern by having the injected polyfill
  script remove itself from the DOM.
* Loaded resources would show up in resource timing APIs. By inlining the
  polyfill script it will not be an external resource load.
* Loaded HTML no longer matches original test resource. This means line numbers
  from test failures will not correspond to original line numbers. We could
  mitigate this by ensuring the injected content does not introduce any new
  newlines to the output, however with the polyfill script inlined this could
  make debugging failures more difficult.

### Adding a canonical resource which includes the polyfill

2. Add the polyfill into testharnessreport.js.
3. Add the polyfill into a new /resources/polyfill.js which could also be
   included by reftests.

We could transparently insert the polyfill into a canonical resource which is
included by most tests, for example `testharnessreport.js` or
`/resources/polyfill.js`. Given that these external resources are expected to
change it should not cause test failures due to the resource.

Advantages:
* Page structure and loaded resources are not modified.

Concerns:
* All tests which don't already include one of these resources (e.g. ref tests)
  would need to be modified to include them. This would represent an ongoing
  maintenance burden to ensure that new tests add these resources and failure to
  do so wouldn't be caught by existing tests. Likely this would result in a
  mixture of tests which support running with polyfills and tests which don't.
* If we later adopt a mechanism which doesn't require explicitly including the
  polyfill resources these additional artifacts may take some time to remove.

### Bootstrap scripts feature

Polyfill resources could be injected using the [bootstrap scripts feature](
https://github.com/w3c/webdriver-bidi/issues/65). This would allow the polyfill
to be transparently injected into tests.

Advantages:
* Polyfill is transparently inserted across all existing tests (including ref
  tests).
* Served files would match original test files.
* As no modification is required, can easily switch mechanisms later.

Concerns:
* Developers would not be able to debug tests served from running `wpt serve` by
  simply loading them in the browser.
* This mechanism is not available yet.

### Extension injection

Similar to the bootstrap scripts feature, we could use an installed extension to
inject the polyfill. As most polyfills would require installing functionality
into the main script isolate, this would likely require injecting a script tag
into the main document.

Advantages:
* Polyfill is transparently inserted across all existing tests (including ref
  tests).
* As no modification is required, can easily switch mechanisms later.

Concerns:
* As we would need to insert a script tag into the document, this would have the
  same DOM observability concerns as the rewriting approach, with the same
  possible mitigations.
* Developers would not be able to debug tests served from running `wpt serve` by
  simply loading them in the browser. They could however install the extension
  which injected the polyfill.
* Mobile browsers have no current support for these kinds of extensions.

## Conclusion

Rewriting the responses from the wpt server is a solution that is available now,
supports all existing tests, and supports testing both via wpt run and by
visiting the served pages from an external browser. While there are
observability concerns they can largely be mitigated by removing the artifacts
added to the page in the injected script. Since injecting a polyfill is not the
default execution path, this is a pragmatic solution that will allow testing
polyfills now and can be modified in the future with minimal changes.
