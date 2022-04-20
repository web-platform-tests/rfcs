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

In order to support testing a polyfill, a runtime argument is added which
when used specifies a single local Javascript file to be injected into
responses. The contents of the file will be added as a classic script (to ensure
the polyfill runs before any tests) automatically to responses with a
`text/html` MIME type after the `doctype` and `html` and `head` opening tags
(if they exist) but before any other tags in the document.

In order to reduce visible effects on running tests, the script is inlined (to
prevent an external resource load which would show up in the resource timing
API) and the script will remove itself from the DOM to ensure that tests which
assume the shape of the DOM is unchanged will still function correctly (e.g.
`document.getElementsByTagName('script')[0]` will still return the first script
in the original test).

E.g. when running `wpt run --inject-script=script.js [browser] [test]` or
`wpt serve --inject-script=script.js` and loading the following HTML page:
```html
<!doctype html>
<div></div>
<script>
/** contents of test **/
</script>
```

The response will be modified to the following:
```html
<!doctype html>
<script>
/** Contents of script.js **/
// Remove the injected script tag from the DOM.
document.currentScript.remove();
</script>
<div></div>
<script>
/** contents of test **/
</script>
```

This transparent injection of the polyfill script allows the injected script to
run before any tests, and on all loaded HTML documents including reftests which
often don't include any script resources without requiring any modification to
existing test files. It also supports testing in a local browser as `wpt serve`
responses are similarly modified.

## Risks

There are a few risks or downsides with the approach that may lead to false
test failures or failure to inject the polyfill.

* While it should not be possible to observe the script removal in the main
  frame, it may be possible to observe it with a MutationObserver on a loading
  subframe.
* Line numbers in failure messages will not match their original source as they
  will include injected content. We could mitigate this in the future by
  stripping newlines and injecting the entire polyfill into a single
  pre-existing line.
* The polyfill is not injected on pages served from python response handlers
  which write directly to the output response stream. This could be supported
  with future modifications.

## Alternatives considered

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
* All tests which don't already include one of these resources (e.g. reftests)
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
* Polyfill is transparently inserted across all existing tests (including reftests).
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
* Polyfill is transparently inserted across all existing tests (including reftests).
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
supports HTML tests, and supports testing both via `wpt run` and by
visiting the served pages from an external browser. While there are
observability concerns they can largely be mitigated by removing the artifacts
added to the page in the injected script. Since injecting a polyfill is not the
default execution path, this is a pragmatic solution that will allow testing
polyfills now and can be modified in the future with minimal changes.
