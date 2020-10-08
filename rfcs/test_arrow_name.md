# RFC 64: Use simple arrow funcs as the default test name

## Summary

For `test(() => assert_true(true))`, use `assert_true(true)` as the default test name rather than `document.title`

## Details

At the moment, we practically require all tests to have a name be explicitly given. This makes testharness.js tests much more verbose than WebKit/Blink's js-test.js. For example, currently in WebKit we have:

```javascript
const shadowHost = document.createElement("div");
const shadowRoot = shadowHost.attachShadow({mode: 'closed'});
const shadowRootChild = shadowRoot.appendChild(document.createElement('div'));

shouldBeFalse('shadowHost.isConnected');
shouldBeFalse('shadowRoot.isConnected');
shouldBeFalse('shadowRootChild.isConnected');
shouldBeFalse('document.contains(shadowHost)');
shouldBeFalse('document.contains(shadowRoot)');
shouldBeFalse('document.contains(shadowRootChild)');
```

In WPT, the shortest way to write this is:

```javascript
test(() => assert_false(shadowHost.isConnected), 'shadowHost.isConnected');
test(() => assert_false(shadowRoot.isConnected), 'shadowRoot.isConnected');
test(() => assert_false(shadowRootChild.isConnected), 'shadowRootChild.isConnected');
test(() => assert_false(document.contains(shadowHost)), 'document.contains(shadowHost)');
test(() => assert_false(document.contains(shadowRoot)), 'document.contains(shadowRoot)');
test(() => assert_false(document.contains(shadowRootChild)), 'document.contains(shadowRootChild)');
```

This is comparatively rather verbose, and has been cited as a reason to not adopt testharness.js tests.

This proposal would make the default test names `assert_false(shadowHost.isConnected)`, etc., rather than whatever the `title` element contains. This would only apply when the test function is an arrow function with no arguments where the body contains no new lines, as determined from `Function.prototype.toString`. As a result, the following becomes possible:

```javascript
test(() => assert_false(shadowHost.isConnected));
test(() => assert_false(shadowRoot.isConnected));
test(() => assert_false(shadowRootChild.isConnected));
test(() => assert_false(document.contains(shadowHost)));
test(() => assert_false(document.contains(shadowRoot)));
test(() => assert_false(document.contains(shadowRootChild)));
```

An [implementation](https://github.com/web-platform-tests/wpt/pull/25853) has already been done.

## Risks

Some existing tests relying on `document.title` for their name could have their name change. However, we have few tests that currently match the above. Running the above implementation against Firefox didn't show any test name changes.

If `Function.prototype.toString` is not sufficiently interoperable then we could end up with different test names across different browsers. However, it is believed that in simple cases, such as those likely to occur on a single line, the risk is minimal.
