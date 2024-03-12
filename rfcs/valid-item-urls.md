# RFC 181 - URLManifestItem.url shouldn't contain non-URL-unit characters

## Summary

Currently we end up storing URLs such as: `/xhr/xmlhttprequest-timeout-worker-aborted.html?aborted immediately after send()`. If you pass this to the [URL parser](https://url.spec.whatwg.org/#concept-url-parser) with a base URL of `http://web-platform.test`, this results in a number of [invalid-URL-unit](https://url.spec.whatwg.org/#invalid-url-unit) [validation errors](https://url.spec.whatwg.org/#validation-error).

We should instead store the output of [URL serialize](https://url.spec.whatwg.org/#concept-url-serializer) ∘ [URL parse](https://url.spec.whatwg.org/#concept-url-parser) of the URL.

## Details

There are various systems at various vendors which expect test identifiers to not contain spaces, and when in principle the test identifier is a URL one might expect it to not contain any spaces.

This is primarily a concern with variants, which vastly more often include spaces, such as the example above.

We do have a few files in WPT which contain spaces:

```
gsnedders@gsnedders-marsha web-platform-tests % find . -name '* *'
./tools/wave/test/WAVE Local.postman_environment.json
./tools/wave/test/WAVE Server REST API Tests.postman_collection.json
./tools/wave/tests/WAVE Local.postman_environment.json
./tools/wave/tests/WAVE Server REST API Tests.postman_collection.json
./css/CSS2/syntax/support/'green block.png
./html/semantics/document-metadata/the-meta-element/pragma-directives/attr-meta-http-equiv-refresh/support/url foo
./html/canvas/offscreen/path-objects/2d.path.roundrect.1.radius.dompoint.single argument.worker.js
./html/canvas/offscreen/path-objects/2d.path.roundrect.1.radius.dompoint.single argument.html
./html/canvas/element/path-objects/2d.path.roundrect.1.radius.dompoint.single argument.html
```

Only the last three of these are test files, which generate three test items.

One question here is whether we should disallow test file paths that contain spaces, or whether we should just escape these in the URL.

## Risks

Any test system that keys off the test URLs will have to deal with URLs changing.

This includes the wptmanifest (i.e., wptrunner's expectation manifest), WebKit (and I believe Blink)'s TestExpectations files and results expectations files.

This may also affect items in wpt-metadata.
