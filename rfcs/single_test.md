# RFC: `single_test()` opt-in for single-page tests

## Summary

Single-page tests are simple testharness.js tests that just do assertions and finally call `done()`. They reduce boilerplate as no `test()` or `async_test()` is needed. Callbacks also need not be wrapped with `step_func()` as any error simply fails the test. See [simple sync test](https://github.com/web-platform-tests/wpt/blob/ded00e006a083cacc108e8a1e92963fe7b15de4e/mediacapture-streams/MediaDevices-SecureContext.html) and [simple async test](https://github.com/web-platform-tests/wpt/blob/87d9cd16b9b4992649dc2ea1ecb3261f163e9184/html/webappapis/timers/negative-setinterval.html) for real examples.

Unfortunately, the rules for single-page tests are subtle and many tests accidentally enter this mode. This leads to less consistent results cross-browser, see [FileAPI example](https://wpt.fyi/results/FileAPI/url/url-format.any.worker.html?run_id=312160003&run_id=306970005&run_id=321820002&run_id=319900004).

Introduce `single_test()` as an explicit opt-in for single-page tests. Once added to the existing ~130 tests, remove the old ways of enabling the mode, which happens inadvertently for up to ~640 tests in some browsers.

Example:
```js
single_test();
assert_false(someCondition);
doSomething();
addEventListener('someevent', () => {
  assert_true(someCondition);
  done();
});
```

The `single_page()` method will optionally take the same `name` and `properties` arguments as other test types.

Acknowledgments:
- jgraham [added single-page tests in 2014](https://github.com/w3c/testharness.js/pull/67)
- zcorpan [proposed the `single_test()` opt-in in 2018](https://github.com/web-platform-tests/wpt/pull/11364)

## Details

The current rules for single-page tests are a bit subtle. They are triggered by one of the following happening _before_ any test is explicitly defined:
- Using any assert method
- Calling the global `done()`
- An uncaught exception or unhandled rejection occurs

A problem with the current setup is that uncaught errors are common in tests not intended to be single-page tests, and manifest as a bogus failing subtest, see [FileAPI example](https://wpt.fyi/results/FileAPI/url/url-format.any.worker.html?run_id=312160003&run_id=306970005&run_id=321820002&run_id=319900004). In fact, this appears to be *much* more common than real single-page tests.

Another issue is that in [dedicated and shared workers, `done()` has to be called](https://web-platform-tests.org/writing-tests/testharness-api.html#determining-when-all-tests-are-complete), and the [generated `done()` for any.js tests](https://github.com/web-platform-tests/wpt/blob/b683b48465900b5585bf08ee4b6c25b219944333/tools/serve/serve.py#L292-L304) is called even if the test script failed. This explains some of the results seen in the following.

### Survey of existing tests

A testharness.js change was prepared to find which conditions were hit: [commit](https://github.com/web-platform-tests/wpt/commit/76dec5fd7efc0d681bea894aa39dbbd5eaff084b) &rarr; [runs](https://wpt.fyi/runs?sha=76dec5fd7efc0d681bea894aa39dbbd5eaff084b&max-count=10) / [results](https://wpt.fyi/results/?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) &rarr; [script](https://gist.github.com/foolip/b630c0432ed1a35f0758dd9d01b1fb1b) &rarr; [raw stats](https://gist.github.com/foolip/7fa72c00a3d355710ee58a6521682db6).

Summary:

<table>
  <tr>
    <th>browser
    <th>assert
    <th>done
    <th>error
  <tr>
    <td>chrome-dev
    <td>20
    <td>93
    <td>16
  <tr>
    <td>chrome-stable
    <td>20
    <td>92
    <td>72
  <tr>
    <td>edge-canary
    <td>18
    <td>94
    <td>69
  <tr>
    <td>edge-dev
    <td>18
    <td>94
    <td>8
  <tr>
    <td>firefox-nightly
    <td>22
    <td>88
    <td>305
  <tr>
    <td>firefox-stable
    <td>22
    <td>88
    <td>360
  <tr>
    <td>safari-preview
    <td>16
    <td>85
    <td>557
  <tr>
    <td>safari-stable
    <td>16
    <td>85
    <td>646
  <tr>
    <th>all
    <th>24
    <th>104
    <th>648
</table>

Most tests will both assert something and eventually call `done()`. The categorization depends on details (timing) of how this was measured and so isn't very interesting, but:
- All 24 "assert" cases were checked and confirmed to be async tests with a `done()` somewhere in the test.
- The "done" cases are sync tests or async tests where all asserts happen right before the `done()` call.

<details>
<summary>All of the "assert" and "done" tests:</summary>

```
assert /html/semantics/document-metadata/the-meta-element/pragma-directives/attr-meta-http-equiv-refresh/allow-scripts-flag-changing-1.html
assert /html/semantics/document-metadata/the-meta-element/pragma-directives/attr-meta-http-equiv-refresh/allow-scripts-flag-changing-2.html
assert /html/semantics/document-metadata/the-meta-element/pragma-directives/attr-meta-http-equiv-refresh/not-in-shadow-tree.html
assert /html/semantics/embedded-content/media-elements/ready-states/autoplay-with-slow-text-tracks.html
assert /html/semantics/embedded-content/the-iframe-element/srcdoc_change_hash.html
assert /html/semantics/embedded-content/the-img-element/update-src-complete.html
assert /html/webappapis/timers/negative-settimeout.html
assert /navigation-timing/nav2_test_navigation_type_backforward.html
assert /preload/download-resources.html
assert /preload/link-header-on-subresource.html
assert /preload/link-header-preload.html
assert /preload/link-header-preload-imagesrcset.html
assert /preload/link-header-preload-nonce.html
assert /preload/onerror-event.html
assert /preload/onload-event.html
assert /preload/preload-csp.sub.html
assert /preload/preload-default-csp.sub.html
assert /preload/preload-with-type.html
assert /preload/single-download-late-used-preload.html
assert /webaudio/the-audio-api/the-analysernode-interface/test-analyser-minimum.html
assert /webaudio/the-audio-api/the-analysernode-interface/test-analyser-output.html
assert /webaudio/the-audio-api/the-analysernode-interface/test-analyser-scale.html
assert /workers/constructors/SharedWorker/URLMismatchError.htm
assert /workers/shared-worker-name-via-options.html
done /content-security-policy/script-src/scripthash-default-src.sub.html
done /content-security-policy/style-src/stylehash-default-src.sub.html
done /css/css-lists/nested-list-with-list-style-type-none.html
done /css/css-shapes/shape-outside/shape-image/gradients/shape-outside-radial-gradient-001.html
done /css/css-shapes/shape-outside/shape-image/gradients/shape-outside-radial-gradient-002.html
done /css/css-shapes/shape-outside/shape-image/gradients/shape-outside-radial-gradient-003.html
done /css/css-shapes/shape-outside/shape-image/gradients/shape-outside-radial-gradient-004.html
done /css/css-shapes/spec-examples/shape-outside-010.html
done /css/css-shapes/spec-examples/shape-outside-011.html
done /css/css-shapes/spec-examples/shape-outside-012.html
done /css/css-shapes/spec-examples/shape-outside-013.html
done /css/css-shapes/spec-examples/shape-outside-014.html
done /css/css-shapes/spec-examples/shape-outside-015.html
done /css/css-shapes/spec-examples/shape-outside-016.html
done /css/css-shapes/spec-examples/shape-outside-017.html
done /css/css-shapes/spec-examples/shape-outside-018.html
done /css/css-shapes/spec-examples/shape-outside-019.html
done /custom-elements/parser/parser-fallsback-to-unknown-element.html
done /dom/nodes/Element-getElementsByTagName-change-document-HTMLNess.html
done /fetch/content-length/content-length.html
done /fetch/corb/script-js-mislabeled-as-html-nosniff.sub.html
done /fetch/corb/script-js-mislabeled-as-html.sub.html
done /fetch/corb/script-resource-with-nonsniffable-types.tentative.sub.html
done /fetch/corb/style-css-mislabeled-as-html-nosniff.sub.html
done /fetch/corb/style-css-mislabeled-as-html.sub.html
done /fetch/corb/style-css-with-json-parser-breaker.sub.html
done /fetch/corb/style-html-correctly-labeled.sub.html
done /fetch/images/canvas-remote-read-remote-image-redirect.html
done /FileAPI/url/multi-global-origin-serialization.sub.html
done /html/browsers/browsing-the-web/navigating-across-documents/source/navigate-child-function-parent.html
done /html/browsers/browsing-the-web/navigating-across-documents/source/navigate-child-src-about-blank.html
done /html/rendering/non-replaced-elements/flow-content-0/dialog-display.html
done /html/rendering/non-replaced-elements/margin-collapsing-quirks/multicol-quirks-mode.html
done /html/rendering/non-replaced-elements/margin-collapsing-quirks/multicol-standards-mode.html
done /html/rendering/non-replaced-elements/tables/table-vspace-hspace.html
done /html/rendering/non-replaced-elements/tables/table-vspace-hspace-s.html
done /html/rendering/non-replaced-elements/the-page/iframe-marginwidth-marginheight.html
done /html/semantics/document-metadata/the-meta-element/pragma-directives/attr-meta-http-equiv-refresh/allow-scripts-flag-changing-1.html
done /html/semantics/document-metadata/the-meta-element/pragma-directives/attr-meta-http-equiv-refresh/allow-scripts-flag-changing-2.html
done /html/semantics/document-metadata/the-meta-element/pragma-directives/attr-meta-http-equiv-refresh/dynamic-append.html
done /html/semantics/document-metadata/the-meta-element/pragma-directives/attr-meta-http-equiv-refresh/remove-from-document.html
done /html/semantics/embedded-content/the-iframe-element/move_iframe_in_dom_01.html
done /html/semantics/embedded-content/the-iframe-element/move_iframe_in_dom_02.html
done /html/semantics/embedded-content/the-iframe-element/move_iframe_in_dom_03.html
done /html/semantics/embedded-content/the-iframe-element/move_iframe_in_dom_04.html
done /html/semantics/embedded-content/the-img-element/data-url.html
done /html/semantics/embedded-content/the-img-element/srcset/avoid-reload-on-resize.html
done /html/semantics/embedded-content/the-img-element/update-src-complete.html
done /html/semantics/forms/autofocus/not-on-first-task.html
done /html/semantics/scripting-1/the-script-element/module/charset-02.html
done /html/semantics/scripting-1/the-script-element/module/error-and-slow-dependency.html
done /html/webappapis/scripting/processing-model-2/window-onerror-parse-error.html
done /html/webappapis/scripting/processing-model-2/window-onerror-runtime-error.html
done /html/webappapis/scripting/processing-model-2/window-onerror-runtime-error-throw.html
done /html/webappapis/timers/negative-setinterval.html
done /html/webappapis/timers/type-long-setinterval.html
done /html/webappapis/timers/type-long-settimeout.html
done /import-maps/imported/parsing-addresses.tentative.html
done /import-maps/imported/parsing-schema.tentative.html
done /import-maps/imported/parsing-scope-keys.tentative.html
done /import-maps/imported/parsing-specifier-keys.tentative.html
done /import-maps/imported/resolving-builtins.tentative.html
done /import-maps/imported/resolving-not-yet-implemented.tentative.html
done /import-maps/imported/resolving-scopes.tentative.html
done /import-maps/imported/resolving.tentative.html
done /infrastructure/browsers/firefox/prefs.html
done /infrastructure/testdriver/file_upload.sub.html
done /mediacapture-streams/MediaDevices-SecureContext.html
done /navigation-timing/nav2_test_document_open.html
done /navigation-timing/nav2_test_document_replaced.html
done /navigation-timing/nav2_test_navigate_within_document.html
done /navigation-timing/nav2_test_navigation_type_reload.html
done /navigation-timing/nav2_test_redirect_chain_xserver_partial_opt_in.html
done /navigation-timing/nav2_test_redirect_server.html
done /navigation-timing/nav2_test_redirect_xserver.html
done /server-timing/navigation_timing_idl.https.html
done /server-timing/resource_timing_idl.https.html
done /webaudio/the-audio-api/the-analysernode-interface/test-analyser-scale.html
done /WebCryptoAPI/derive_bits_keys/ecdh_bits.https.any.worker.html
done /WebCryptoAPI/derive_bits_keys/ecdh_keys.https.any.worker.html
done /WebCryptoAPI/derive_bits_keys/hkdf.https.any.worker.html?1001-2000
done /WebCryptoAPI/derive_bits_keys/hkdf.https.any.worker.html?1-1000
done /WebCryptoAPI/derive_bits_keys/hkdf.https.any.worker.html?2001-3000
done /WebCryptoAPI/derive_bits_keys/hkdf.https.any.worker.html?3001-last
done /WebCryptoAPI/derive_bits_keys/pbkdf2.https.any.worker.html?1001-2000
done /WebCryptoAPI/derive_bits_keys/pbkdf2.https.any.worker.html?1-1000
done /WebCryptoAPI/derive_bits_keys/pbkdf2.https.any.worker.html?2001-3000
done /WebCryptoAPI/derive_bits_keys/pbkdf2.https.any.worker.html?3001-4000
done /WebCryptoAPI/derive_bits_keys/pbkdf2.https.any.worker.html?4001-5000
done /WebCryptoAPI/derive_bits_keys/pbkdf2.https.any.worker.html?5001-6000
done /WebCryptoAPI/derive_bits_keys/pbkdf2.https.any.worker.html?6001-7000
done /WebCryptoAPI/derive_bits_keys/pbkdf2.https.any.worker.html?7001-8000
done /WebCryptoAPI/derive_bits_keys/pbkdf2.https.any.worker.html?8001-last
done /webmessaging/without-ports/020.html
done /webmessaging/without-ports/021.html
done /webmessaging/with-ports/020.html
done /webmessaging/with-ports/021.html
done /workers/constructors/SharedWorker/URLMismatchError.htm
done /workers/interfaces/WorkerGlobalScope/close/incoming-message.html
done /workers/interfaces/WorkerGlobalScope/close/setInterval.html
done /workers/interfaces/WorkerGlobalScope/close/setTimeout.html
done /workers/semantics/multiple-workers/004.html
done /workers/semantics/navigation/001.html
done /workers/shared-worker-name-via-options.html
```
</details>

The "error" case is usually hit when the test tries to use some API that isn't supported in the browser under test. That's why there are more of these in stable browsers. Most of these tests aren't intended to be single-page tests.

<details>
<summary>A few tests hit the "assert" or "done" condition in one browser but the "error" condition in another:</summary>

- [/css/css-shapes/spec-examples/shape-outside-018.html](https://wpt.fyi/results/css/css-shapes/spec-examples/shape-outside-018.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/css/css-shapes/spec-examples/shape-outside-018.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/navigation-timing/nav2_test_document_open.html](https://wpt.fyi/results/navigation-timing/nav2_test_document_open.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/navigation-timing/nav2_test_document_open.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/navigation-timing/nav2_test_navigate_within_document.html](https://wpt.fyi/results/navigation-timing/nav2_test_navigate_within_document.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/navigation-timing/nav2_test_navigate_within_document.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/navigation-timing/nav2_test_navigation_type_backforward.html](https://wpt.fyi/results/navigation-timing/nav2_test_navigation_type_backforward.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/navigation-timing/nav2_test_navigation_type_backforward.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/navigation-timing/nav2_test_navigation_type_reload.html](https://wpt.fyi/results/navigation-timing/nav2_test_navigation_type_reload.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/navigation-timing/nav2_test_navigation_type_reload.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/navigation-timing/nav2_test_redirect_chain_xserver_partial_opt_in.html](https://wpt.fyi/results/navigation-timing/nav2_test_redirect_chain_xserver_partial_opt_in.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/navigation-timing/nav2_test_redirect_chain_xserver_partial_opt_in.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/navigation-timing/nav2_test_redirect_server.html](https://wpt.fyi/results/navigation-timing/nav2_test_redirect_server.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/navigation-timing/nav2_test_redirect_server.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/navigation-timing/nav2_test_redirect_xserver.html](https://wpt.fyi/results/navigation-timing/nav2_test_redirect_xserver.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/navigation-timing/nav2_test_redirect_xserver.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/server-timing/navigation_timing_idl.https.html](https://wpt.fyi/results/server-timing/navigation_timing_idl.https.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/server-timing/navigation_timing_idl.https.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/webaudio/the-audio-api/the-analysernode-interface/test-analyser-minimum.html](https://wpt.fyi/results/webaudio/the-audio-api/the-analysernode-interface/test-analyser-minimum.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/webaudio/the-audio-api/the-analysernode-interface/test-analyser-minimum.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/webaudio/the-audio-api/the-analysernode-interface/test-analyser-output.html](https://wpt.fyi/results/webaudio/the-audio-api/the-analysernode-interface/test-analyser-output.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/webaudio/the-audio-api/the-analysernode-interface/test-analyser-output.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/webaudio/the-audio-api/the-analysernode-interface/test-analyser-scale.html](https://wpt.fyi/results/webaudio/the-audio-api/the-analysernode-interface/test-analyser-scale.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/webaudio/the-audio-api/the-analysernode-interface/test-analyser-scale.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/WebCryptoAPI/derive_bits_keys/ecdh_bits.https.any.worker.html](https://wpt.fyi/results/WebCryptoAPI/derive_bits_keys/ecdh_bits.https.any.worker.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/WebCryptoAPI/derive_bits_keys/ecdh_bits.https.any.worker.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/WebCryptoAPI/derive_bits_keys/ecdh_keys.https.any.worker.html](https://wpt.fyi/results/WebCryptoAPI/derive_bits_keys/ecdh_keys.https.any.worker.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/WebCryptoAPI/derive_bits_keys/ecdh_keys.https.any.worker.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/workers/constructors/SharedWorker/URLMismatchError.htm](https://wpt.fyi/results/workers/constructors/SharedWorker/URLMismatchError.htm?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/workers/constructors/SharedWorker/URLMismatchError.htm?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/workers/semantics/multiple-workers/004.html](https://wpt.fyi/results/workers/semantics/multiple-workers/004.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/workers/semantics/multiple-workers/004.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
- [/workers/shared-worker-name-via-options.html](https://wpt.fyi/results/workers/shared-worker-name-via-options.html?run_id=310440006&run_id=297430002&run_id=287730001&run_id=297450002&run_id=314200005&run_id=306940004&run_id=312100003&run_id=314200002) ([master](https://wpt.fyi/results/workers/shared-worker-name-via-options.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011))
</details>

[test-analyser-minimum.html](https://wpt.fyi/results/webaudio/the-audio-api/the-analysernode-interface/test-analyser-minimum.html?run_id=297370001&run_id=319840011&run_id=314120012&run_id=306900006&run_id=291680008&run_id=287670008&run_id=275610007&run_id=319850011) is a good example of a test that could be impacted by a change.

Conclusions:
- There aren't very many single-page tests right now
- The accidents far outnumber the intentional uses

## Risks

Single-page tests are intended to have as little boilerplate as possible, based on feedback from Gecko developers familiar with [Mochitest](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Mochitest), see [test example](https://github.com/mozilla/gecko-dev/blob/01c6764830acaabafeec509f5512f8ef564d6964/dom/tests/mochitest/bugs/test_protochains.html). By requiring both `simple_test()` and `done()` even for sync tests where SimpleTest.js requires neither, some Gecko engineers might prefer to use Mochitest instead of WPT.

Updates to testharness.js haven't always been made together with the test updates in all browser engine repos. If this is still the case, the testharness.js changes have to be made first and synced downstream before any test changes are made.

It's difficult to with certainty identify all single-page tests that exist and need to be updated. They will be identified by custom test runs as above and by searching for uses of `done()` in the repo. Finally, results before and after the changes need to be carefully compared.

The new failure mode of tests that were accidentally single-page tests should be no less obvious than before. Some tests may need to be updated to ensure this.

## Alternatives considered

### Just fix the tests

The accidental single-page tests could be fixed, often by wrapping setup code in `setup()`. However, without at least removing the "error" trigger, such tests just look like they have differently named subtests, which is sometimes OK. That makes any tooling to identify the problem from results noisy, and such tests would continue to accumulate.

### Remove only the "error" trigger

jgraham proposed [Don't assume file_is_test when there's a top-level non-assert error](https://github.com/web-platform-tests/wpt/pull/11024) &rarr; [runs](https://wpt.fyi/runs?sha=6137405db45618d2304e645bb26893490bb20a84&max-count=10) / [results](https://wpt.fyi/results/?run_id=318150001&run_id=303190007&run_id=318140006). The [FileAPI example](https://wpt.fyi/results/FileAPI/url/url-format.any.worker.html?run_id=318150001&run_id=303190007&run_id=318140006) turned into a timeout instead, still masking the problem.

foolip tried [Remove the error_handler trigger for single-page tests](https://github.com/web-platform-tests/wpt/commit/9cfe47650fd86a039effe162ab9ea039d329dbb3) &rarr; [runs](https://wpt.fyi/runs?sha=9cfe47650fd86a039effe162ab9ea039d329dbb3&max-count=10) / [results](https://wpt.fyi/results/?run_id=319960008&run_id=316100006&run_id=318120011). The [FileAPI example](https://wpt.fyi/results/FileAPI/url/url-format.any.worker.html?run_id=319960008&run_id=316100006&run_id=318120011) is _still_ no good, now there's a bogus passing subtest instead. As mentioned above, this is because `done()` is called in generated code.

With additional tweaking one can probably ensure that an uncaught error is treated as a harness error, solving most of the problem. However, some minor warts would remain:
- A test would have reach an assert or `done()` before it's known to be a single-page test.
- Therefore, one couldn't rely on an exception to fail the test, with behavior changing after the first assert. This is subtle, and to avoid a harness error a bogus assert or `setup()` wrapping would be needed.

Note: not all harness errors can be avoided, but they are often a sign of a test problem, and so avoiding them where possible help highlight those problems.

### Also remove the "assert" trigger

This would leave `done()` as the only opt-in, and the behavior would not change on the first assert. However, any failing assert would turn into a harness error, which is no good. Fixing it with an early opt-in amounts to the `single_test()` proposal.

### Add `single_page()` as optional

Keeping `done()` but adding `single_page()` as optional would avoid boilerplate in a small number of existing tests that pass consistently. However, it would only be sound for tests that have no asserts and could never throw exceptions or reject promises. This is too small a niche of tests to warrant more complex rules for single-page tests.
