# RFC 242: Wide-Gamut and HDR Screenshot Reftests

## Summary

This RFC adds opt-in `reftest-color-space` metadata for WPT reftests. For an opted-in test, `wptrunner` requests a PNG screenshot in the requested output color space through the executor and compares the decoded samples without converting them to 8-bit sRGB. The same request applies to the root test and its complete reference graph. Tests without the metadata keep the current behavior.

The initial implementation targets `display-p3` / `image/png`. Other wide-gamut and HDR combinations can be enabled after the WebDriver specifications define their encoded output and WPT can decode and compare it without losing the values under test.

The protocol extensions for classic WebDriver and WebDriver BiDi are outside the scope of this RFC. They should use the same capture requirements. The classic WebDriver proposal is tracked in [webdriver#1978](https://github.com/w3c/webdriver/issues/1978). The BiDi proposal is tracked in [webdriver-bidi#1114](https://github.com/w3c/webdriver-bidi/issues/1114).

## Background

The current WebDriver screenshot path is effectively limited to sRGB SDR. An sRGB-only capture can hide a rendering bug: if the test is incorrectly clamped to sRGB and the reference is also converted to sRGB during capture, the two can compare equal. For HDR, tone mapping or clamping to SDR removes the values under test entirely.

JPEG XL is the first concrete consumer. The tests to automate cover wide-gamut SDR content in Adobe RGB, Display-P3, and Rec.2020, and HDR content in PQ and HLG. Gain maps remain a follow-up testing target. References must also exercise values outside sRGB or SDR, since capturing an sRGB-only reference into a wider output space does not validate wide-gamut or HDR rendering. JPEG XL decoding is not necessarily bit-exact, so some tests will also need tightly bounded fuzzy comparison. The proposed comparison rules are not specific to JPEG XL.

## Details

### Authoring and reference graph

A reftest opts in by declaring the requested output color space on the root test:

```html
<meta name="reftest-color-space" content="display-p3">
```

The value uses the identifier defined by the WebDriver specifications. WPT does not define aliases or separate color-space semantics. Planned outputs are `srgb` and `display-p3`, with `rec2100-pq` as a future HDR output.

These are screenshot output spaces, not the color spaces of the input content. Separate Adobe RGB, SDR Rec.2020, and HLG outputs are deferred. The corresponding input tests remain in scope. Display-P3 output alone will not cover all of them.

The metadata is valid only on the root test and applies to every match and mismatch reference in its reference graph. Reference files do not override the root request. This allows the same reference file to be reused by tests that request different output color spaces.

The manifest stores the requested color space on the root reftest item so the value is available to the runner before execution. Unknown identifiers, malformed values, and duplicate metadata are authoring errors. A valid request that the browser or driver does not support is a capture error instead.

`wptrunner` uses this value to select a capture path that meets the capture contract. If that path requires different browser launch settings, the browser integration provides them through `Browser.settings(test)`. The runner starts or restarts the browser with those settings. The executor handles setup within the current session. Any setup required before navigation must finish before loading the test or any reference. The same settings must apply to the test and all references. Before a test with different requirements reuses the environment, the runner and executor must remove or replace changed settings, including after errors.

### Requesting screenshot output

For opted-in tests, `wptrunner` passes the requested color space to the executor. The executor may use classic WebDriver or WebDriver BiDi. The current BiDi proposal would send `colorSpace` and `format: { type: "image/png" }` to `browsingContext.captureScreenshot`. The APIs and capture requirements for these extensions are still under discussion in both protocols. The encoded image type is fixed by the initial implementation and is not exposed as test metadata.

Support is defined for a `colorSpace` / `image/png` pair. WPT enables a value only after the WebDriver specifications define the corresponding PNG output and the WPT decode path supports it. An executor needs only one capture path that meets these requirements. Support for PNG alone does not imply support for every color space. Other encoded image types require a follow-up RFC together with the corresponding protocol and comparison work.

### Capture contract

For each enabled `colorSpace` / `image/png` pair, the WebDriver specifications define enough about the returned PNG for WPT to validate and interpret it. This includes the rendering requirements, required bit depth, color signaling, and sample interpretation. This RFC refers to those requirements as the **capture contract**. WPT consumes the contract; it does not define the HDR value semantics or the PNG representation.

This RFC proposes using the requested output gamut as the target for rendering the capture. The browser may reuse rendering data that preserves the required values. Otherwise, it must perform the rendering needed for the requested target. Converting pixels after those values have been lost is not sufficient. The requested space need not match the physical display.

For future `rec2100-pq` screenshots, the capture contract must map SDR reference white to 203 cd/m² (203 nits) on the PQ scale, following [CSS Color HDR](https://www.w3.org/TR/css-color-hdr-1/#compositing). This defines the reference white of the returned image, not its maximum luminance or the brightness of the attached display.

The runner reports `ERROR` when the required rendering or capture mode is unavailable, a request is rejected, a fallback is explicitly reported, or a PNG does not satisfy the capture contract. For these capture errors, it logs the cause and does not compare the images. Valid captures use the normal match or mismatch comparison to determine `PASS` or `FAIL`. Incorrect rendering is not reclassified as unsupported.

Future gain-map tests require defined rendering conditions, including the target HDR headroom used when applying the gain map. The test and all references must use the same target headroom. Choosing a PNG color space or a PQ reference white does not set this condition. An SDR gain-map test would use a linear headroom of 1 (0 stops) when applying the gain map, not merely convert a later HDR result to SDR. The capture contract must define how the required condition is established before those tests are enabled. This may use capture settings or a defined mechanism in the test and reference pages.

### Comparison and `wptrunner`

The test and reference PNGs are validated and decoded using the same capture contract. Decoding preserves the stored samples and does not convert them to sRGB, reduce their bit depth, gamut-map them, or tone-map them. Comparison uses decoded channel values rather than encoded PNG bytes, since different PNG encodings can represent the same samples.

Exact comparison is sufficient for synthetic fixtures and deterministic paths. For opted-in tests, this RFC proposes the following fuzzy comparison for full-range integer RGB or RGBA samples. Both captures must satisfy the same color-space and bit-depth requirements. The initial comparison uses the same viewport, device scale, and capture rectangle for the test and reference. Both PNGs must have the pixel dimensions required by those settings. Unexpected or differing dimensions are capture errors, reported as `ERROR`, and cannot satisfy a mismatch reference.

At each pixel, divide the largest absolute RGB channel difference by `2^b - 1`, where `b` is the bit depth. Compare the largest value across the image with `maxChannelDifference`. Count each pixel where any compared RGB channel differs once, before applying the tolerance, and compare that count with `maxDifferentPixels`. One sample step is `1/255` for 8-bit samples and `1/65535` for 16-bit samples.

Alpha must match exactly. Treat a missing alpha channel as fully opaque. Ignore RGB only where both alpha values are zero. Otherwise, compare the stored RGB without multiplying it by alpha. An alpha difference makes the images unequal regardless of the RGB tolerance.

This RFC proposes extending the existing `fuzzy` annotation with new keys for opted-in comparisons:

```html
<meta name="fuzzy"
      content="maxChannelDifference=2/255;maxDifferentPixels=100">
```

Both new keys must be present and explicitly named. They define inclusive upper limits rather than the ranges used by the existing format. For a match comparison, smaller differences are acceptable, including an exact match. Unnamed values are allowed only in the existing format. The example illustrates syntax, not recommended tolerances.

`maxChannelDifference` accepts a value from 0 to 1, written as a finite decimal without an exponent or as `p/q`, where `p` is a nonnegative integer and `q` is a positive integer. For example, `2/255` allows two steps in an 8-bit sample. Interpret both forms exactly, without rounding to binary floating point. Express a decimal as an exact fraction. For a threshold `p/q`, the largest integer channel difference `d` is allowed when `d * q <= p * (2^b - 1)`. `maxDifferentPixels` is a nonnegative integer.

Use the annotation on the page that owns the reference link. An optional reference URL followed by a colon selects a particular reference and takes precedence over a general annotation. Each metadata source may provide only one applicable entry per selector. Mixed old and new keys, duplicate or unknown keys, and invalid values are authoring errors.

Vendors use the existing `fuzzy` property in the root test's `.ini` metadata. The new format uses the same two keys and units as HTML and retains the existing reference and comparison-pair selectors. Evaluate `.ini` conditions before selecting an entry. Within `.ini`, a comparison-pair entry takes precedence over a reference entry, which takes precedence over a general entry. An applicable `.ini` entry takes precedence over HTML. Select the entry in this order before checking its format, and take both limits from that entry.

For a comparison whose root declares `reftest-color-space`, the selected entry must use the new keys. A selected entry in the existing format is a metadata error; do not skip it to use a lower-priority entry or reinterpret its values. Comparisons without `reftest-color-space` keep the existing format and behavior. Selecting an entry with the new keys for such a comparison is a metadata error. If no entry applies to an opted-in comparison, both limits are zero.

Under this proposal, a shared reference page cannot provide both formats for the same selector. Authors can use separate reference pages when different root tests need different formats for that selector.

WPT lint checks HTML annotation syntax and reports entries in the existing format in a root that declares `reftest-color-space`. The runner validates `.ini` entries when loading and evaluating metadata, and checks the selected format against the root's comparison mode. Invalid selected entries must not fall back to another source or to exact comparison. Report a test-specific metadata error as `ERROR`; a file-level parse error remains a metadata loading error.

After validating capture dimensions, the images count as equal when alpha matches and neither limit is exceeded. If all RGB values are ignored, both measurements are zero. A match reference requires the images to be equal under these rules. A mismatch reference requires them to be unequal. These are numeric, not perceptual, differences. Tolerances must be justified for each test and output. An sRGB tolerance is not automatically suitable for P3 or PQ, and a PQ sample difference is not a fixed difference in nits. The metric and alpha policy need validation with fixtures before use. Tests needing an alpha tolerance or a different comparison metric require follow-up work. This may include gain-map tests.

Fuzzy comparison does not define which tone-mapping results are correct. Where the rendering rules allow different tone-mapping choices, tests must not require one implementation's current curve. A tolerance must not be chosen only because it makes current browser outputs pass. If no reference and tolerance can be justified, another automated check may be used if it respects those choices. Cases with neither option remain manual.

`wptrunner` passes the resolved request to the executor and preserves the original PNG for validation and diagnostics. The requested color space is part of the screenshot cache key so captures of the same URL in different color spaces do not collide. Any other rendering conditions that affect the capture, including target headroom when supported, must also be part of that key. A vendor runner may use a different capture path as long as it produces output satisfying the same capture contract and compares the same decoded samples.

On a rendering mismatch or invalid capture, the runner keeps the available original PNGs as artifacts. A derived sRGB or tone-mapped preview may be provided for debugging, but it is not used for comparison.

## Risks

- **Decoder support.** Existing Python image libraries and WPT code paths may reduce precision, convert to sRGB, discard color signaling, or tone-map. The color-preserving path needs targeted fixtures for these failures.
- **Capture validation.** Tests of the capture implementation must check known pixel values through rendering, capture, and decoding, not only the PNG color profile. The fixtures should also include a mismatch pair whose colors differ in Display-P3 but become equal after a defined sRGB clipping conversion.
- **Protocol and implementation dependency.** WPT cannot enable a color space until the WebDriver specifications define the corresponding PNG output and at least one browser and runner can produce and decode it. `display-p3` is the intended first end-to-end path; PQ remains dependent on protocol and implementation feasibility.
- **Fuzzy comparison.** Exact comparison is likely too strict for broad JPEG XL coverage, but an underspecified higher-precision tolerance would produce inconsistent results across runners.
- **Reference quality.** A reference containing only sRGB or SDR values makes an opted-in test ineffective.
- **Cost and debugging.** Higher-bit-depth PNGs require more storage and decode time and may not display correctly in ordinary image viewers. Capture paths that require new launch settings also incur the cost of restarting the browser when those settings change.
- **Downstream consumers.** Vendor harnesses, dashboards, and artifact systems need to handle the new metadata, capture errors, and image artifacts. A harness that ignores `reftest-color-space` may compare sRGB captures and report a false pass. A harness that parses `fuzzy` itself needs to handle the new keys.

## References

- WPT tracking issue: <https://github.com/web-platform-tests/wpt/issues/44320>
- Chromium tracking issue: <https://crbug.com/483413433>
- Interop JPEG XL scoping: <https://github.com/web-platform-tests/interop-jpegxl/issues/2>
- Interop JPEG XL manual HDR tests: <https://github.com/web-platform-tests/interop-jpegxl/issues/5>
- Classic WebDriver proposal: <https://github.com/w3c/webdriver/issues/1978>
- WebDriver BiDi proposal: <https://github.com/w3c/webdriver-bidi/issues/1114>
- WebDriver BiDi `captureScreenshot`: <https://w3c.github.io/webdriver-bidi/#command-browsingContext-captureScreenshot>
- CSS Color 4: <https://www.w3.org/TR/css-color-4/>
- CSS Color HDR Module Level 1: <https://www.w3.org/TR/css-color-hdr-1/>
- Reftest documentation: <https://web-platform-tests.org/writing-tests/reftests.html>
