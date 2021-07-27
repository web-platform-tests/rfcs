# RFC 87: `testdriver` Extend the mechanisms for giving browsing contexts ids.

## Summary

Allow contexts to be identified using either a `testdriver_id`
property on the (window) global, or a `uuid` query parameter, either
on the container `src` attribute (for nested browsing contexts) or the
actual resource URL.

## Details

When interacting with either browsing contexts or scripting realms
(hereafter: contexts) other than the test window, it's necessary to
have an identifier for the context that's available both on the js
side and can also be used by WebDriver to identify the context in
question.

Because of the limitations of the WebDriver-based testdriver setup, we
don't have an existing UUID for each context which could be reused for
this purpose. Instead the `testdriver-extra.js` file in wptrunner
has a `get_window_id` function that takes a `WindowProxy` object,
looks for a `__wptrunner_id` property, if not present sets it to a new
UUID, and then returns the value. Then on the wptrunner side, we use
WebDriver to search the tree of open contexts, executing script in
each one to find the window with that property set.

This setup works OK for cases where the cross-origin policy allows
setting the property. We also support the case where the non-test
window includes testdriver.js and can postMessage a request to the
top-level window to initiate an action; with some work this allows
supporting any case where it's possible to get a js handle between the
test window and the top-level window. However we have no support for
noopener cases or COOP headers, or other cases where it's not possible
to pass messages between windows using js alone.

To address these use cases, we propose allowing the use of a `uuid`
query parameter on URLs, which will provide an alternate identifier
for the window that loads the URL.

In addition we propose updating the `__wptrunner_id` property name to
`testdriver_id`, and making it a documented part of the harness
setup. This will allow the default implementation of testdriver.js to
refer to the property directly rather than it being part of the
wptrunner implementation.

testdriver will get a new function `get_context_id(ctx)`, with the
following behaviour:
* If `ctx` is a string, return `ctx`.
* If ctx is a WindowProxy with a readable `location.href` which parses
  as a URL and has a `uuid` query parameter, return the UUID parameter.
* If ctx is a WindowProxy and it has a `testdriver_id` attribute,
  return the value of the attribute, or if it doesn't have such an
  attribute, but the Window is writable from the current context, set
  `testdriver_id` to a new id and return that.
* If `ctx` is a nested browsing context container with a `src`
  attribute (for `iframe`, `frame`, `embed`) or `data` attribute (for
  `object`) and the attribute value parses as a URL with a uuid
  query parameter, return the uuid parameter.

On the wptrunner side, we keep the same behaviour of searching through
the tree of browsing contexts, but in addition we parse `uuid` query
strings out of the attribute values of nested context containers and
the URLs of loaded documents, and match those against the target URL.

In this system a specific context may have more than one associated id
(if the various ways of setting it differ), but we can use any
associated id to get to the browsing context.

For noopener cases, we can now provide a uuid in the URL and use that
to invoke testdriver actions on the remote context:

```
// Use the server subn to generate a UUID
let uuid = "{{uuid()}}"
open(`child.html?uuid=${uuid}`)
// [...] stuff to ensure child is loaded etc.
// In this case the context identifier is used directly
test_driver.delete_all_cookies(uuid)
```

We can also access a cross-origin-nested context:
```
<iframe src=https://{{host}}:{{ports[https][1]}}/file.html?uuid={{uuid()}}></iframe>
<script>
onload = () => {
  let frame = frames[0];
  test_driver.delete_all_cookies(frame)
}
</script>
```

TODO: How should this extend to other resources such as worker scripts. For
current testdriver functionality it doesn't make much sense to run in
a worker, but once we have cross-context messaging we might want to
send messages to a worker we can't access via js.

## Risks

This may clash with existing use of a uuid parameter on URLs. A full
CI runs should be done to avoid unexpected changes to the test
results.

The system of having multiple ids for a given context is complex. In
particular there's a risk that once we have WebDriver implementations
that natively support providing an id to each context
(e.g. WebDriver-BiDi) we won't want to support this kind of slightly
hacky approach, but tests may depend on the implementation
details. That was the original reason for `__wptrunner_id` being
clearly an implementation detail.

## References

[PR 29803](https://github.com/web-platform-tests/wpt/pull/29803)
contains a prototype implementation of this.
