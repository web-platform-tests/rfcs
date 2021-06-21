# RFC 86: Add `fetch_tests_from_prefixed_local_storage` method

## Summary

Add `fetch_tests_from_prefixed_local_storage` method that fetches test results from another page via local storage and `/common/PrefixedLocalStorage.js`.
This is similar to `fetch_tests_from_window`, but is designed for cases where direct communication channels including `postMessage` are not available, like a `Window` opened by `window.open(url, "noopener")`.

This is primarily for the use in Back-foward cache WPTs (see [wpt/pull/28950](https://github.com/web-platform-tests/wpt/pull/28950) for actual `testharness.js` changes and use cases), but can be also used for other tests involving isolated Windows or top-level navigations.

## Details

Testing things requiring isolated windows is not well supported by the WPT frameworks.
For example, when `A.html` calls `window.open('B.html', 'noopener')` and tests something on `B.html`, `B.html` sends data to `A.html` and `A.html` calls `testharness.js` APIs like `assert_equals()` on the data, like:

A.html:
```
async_test(t => {
  window.open(prefixedLocalStorage.url('B.html'), '_blank', 'noopener');
  wait_for_result();
  assert_equals(localStorage.getItem('result'), 'expected');
});
```

B.html:
```
localStorage.setItem('result', doSomethingAndGetResult());
```

The problems here are:

- Test logic often spreads across multiple HTMLs, which can make the logic and its failure mode unclear.
- we need non-`postMessage` communication mechanisms (which tend to be one-off; see below for existing examples).

Instead, this RFC proposes 

- Writing tests using `testharness.js` APIs on isolated Windows (`B.html` in the example above), and
- Communicating test results within a new API `fetch_tests_from_prefixed_local_storage` in `testharness.js` from the isolated Windows to the main test Window (`A.html` in the example).

A.html:
```
const prefixedLocalStorage = new PrefixedLocalStorageTest();
fetch_tests_from_prefixed_local_storage(prefixedLocalStorage);
window.open(prefixedLocalStorage.url('B.html'), '_blank', 'noopener');
```

B.html:
```
async_test(t => {
  assert_equals(doSomethingAndGetResult(), 'expected');
});
```

By doing so,

- More test logic can be centralized to `B.html`, especially when the most of the test logic can be done in `B.html` and `window.open() + noopener` is needed just to create a separate browsing context (this is the case for most of the BFCache tests).
- Communication implementation is encapsulated within `testharness.js`, instead of within individual tests.

## Risks

We might want to change the implementations within `fetch_tests_from_prefixed_local_storage`, when

* More use cases are to be supported (like cross-origin Windows)
* `localStorage` spec or implementation is changed

This kind of risks is somehow inherent regardless of communication methods we use, because we anyway want communications between isolated Windows, which might be blocked by security/privacy efforts.

Changes needed will be anyway small, because I expect

* We only have to change the implementation within `fetch_tests_from_prefixed_local_storage` and boilerplate-like code on its caller side, (hopefully) not the test contents, given that the APIs supported in the test contents are stable `testharness.js` APIs.
* The number of tests using `fetch_tests_from_prefixed_local_storage` will be limited.

## Existing communication helpers

Existing tests (WPT or Chromium, mainly those with `window.open()` + `noopener`) uses several communication mechanisms/helpers for this purpose.

- localStorage
    - [/wpt/common/PrefixedLocalStorage.js](https://github.com/web-platform-tests/wpt/blob/master/common/PrefixedLocalStorage.js)
        - e.g. used by [/wpt/html/browsers/windows/auxiliary-browsing-contexts/opener-noopener.html](https://github.com/web-platform-tests/wpt/blob/master/html/browsers/windows/auxiliary-browsing-contexts/opener-noopener.html)
- sessionStorage
    - [http/tests/navigation/resources/back-to-get-after-post-helper.html](https://source.chromium.org/chromium/chromium/src/+/main:third_party/blink/web_tests/http/tests/navigation/resources/back-to-get-after-post-helper.html)
- Fetch API + server-side stash
    - [/wpt/scroll-to-text-fragment/stash.js](https://github.com/web-platform-tests/wpt/blob/master/scroll-to-text-fragment/stash.js)
    - [/wpt/html/browsers/windows/resources/window-name-stash.py](https://github.com/web-platform-tests/wpt/blob/master/html/browsers/windows/resources/window-name-stash.py)
    - [/wpt_internal/prerender/resources/key-value-store.py](third_party/blink/web_tests/wpt_internal/prerender/resources/key-value-store.py)

## Alternatives Considered

### Fetch API + server-side stash instead of `localStorage`

- pros: More robust than `localStorage`: Fetch API will be less affected by efforts to block communications between windows, and works even in cross-origin cases.
- cons: Fetch API is async, and pages with open fetch requests are not BFCache-eligible in Chromium. Therefore, a little more coordination is needed to ensure the fetch requests for accessing stash are closed before navigation.

### BroadcastChannel instead of `localStorage`

- cons: Pages with open BroadcastChannels is not BFCache-eligible in Chromium.
- cons: BroadcastChannel is not supported in Safari.

### Introduce a new class of tests that drive the browser from the outside

Instead of writing tests and passing data within JavaScript, introduce a new class of tests and its infrastructure controling the browser "from the outside", much more like WebDriver tests.

- cons: Design and implementation costs are much higher.
- It would be preferable to create a design that covers a broader use cases like navigation, BFCache, prerendering, etc., but right now I don't have clear ideas (both on design/implementation and use cases).
