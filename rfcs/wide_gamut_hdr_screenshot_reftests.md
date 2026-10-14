# RFC 242: Wide-Gamut and HDR Screenshot Reftests

## Summary

This RFC adds opt-in `reftest-color-space` metadata for WPT reftests. For an opted-in test, `wptrunner` requests a PNG screenshot in the requested output color space through the executor and compares the decoded samples without converting them to 8-bit sRGB. The same request applies to the root test and its complete reference graph. Tests without the metadata keep the current behavior.

The initial implementation targets `display-p3` / `image/png`. Other wide-gamut and HDR combinations can be enabled after the WebDriver specifications define their encoded output and WPT can decode and compare it without losing the values under test.

The protocol extensions for classic WebDriver and WebDriver BiDi are outside the scope of this RFC. They should use the same capture requirements. The classic WebDriver proposal is tracked in [webdriver#1978](https://github.com/w3c/webdriver/issues/1978). The BiDi proposal is tracked in [webdriver-bidi#1114](https://github.com/w3c/webdriver-bidi/issues/1114).

## Background

The current WebDriver screenshot path is effectively limited to sRGB SDR. An sRGB-only capture can hide a rendering bug: if the test is incorrectly clamped to sRGB and the reference is also converted to sRGB during capture, the two can compare equal. For HDR, tone mapping or clamping to SDR removes the values under test entirely.

JPEG XL is the first concrete consumer. The expected wide-gamut SDR targets are Adobe RGB, Display-P3, and Rec.2020; the first HDR targets are PQ and HLG. Gain maps are a later target while encoder support remains limited. References must also exercise values outside sRGB or SDR, since capturing an sRGB-only reference into a wider output space does not validate wide-gamut or HDR rendering. JPEG XL decoding is not necessarily bit-exact, so some tests will also need tightly bounded fuzzy comparison.

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

### Requesting screenshot output

For opted-in tests, `wptrunner` passes the requested color space to the executor. The executor may use classic WebDriver or WebDriver BiDi. The current BiDi proposal would send `colorSpace` and `format: { type: "image/png" }` to `browsingContext.captureScreenshot`. The APIs and capture requirements for these extensions are still under discussion in both protocols. The encoded image type is fixed by the initial implementation and is not exposed as test metadata.

Support is defined for a `colorSpace` / `image/png` pair. WPT enables a value only after the WebDriver specifications define the corresponding PNG output and the WPT decode path supports it. An executor needs only one capture path that meets these requirements. Support for PNG alone does not imply support for every color space. Other encoded image types require a follow-up RFC together with the corresponding protocol and comparison work.

### Capture contract

For each enabled `colorSpace` / `image/png` pair, the WebDriver specifications define enough about the returned PNG for WPT to validate and interpret it. This includes the required bit depth, color signaling, and sample interpretation. This RFC refers to those requirements as the **capture contract**. WPT consumes the contract; it does not define the HDR value semantics or the PNG representation.

For future `rec2100-pq` screenshots, the capture contract must map SDR reference white to 203 cd/m² (203 nits) on the PQ scale, following [CSS Color HDR](https://www.w3.org/TR/css-color-hdr-1/#compositing). This defines the reference white of the returned image, not its maximum luminance or the brightness of the attached display.

A rejected request, an explicitly reported fallback, or a PNG that does not satisfy the capture contract is reported as a capture error and is not compared. The log identifies the cause. A capture error remains distinct from `FAIL`, which means that two valid captures differed. In particular, the runner does not compare fallback sRGB or tone-mapped SDR pixels.

### Comparison and `wptrunner`

The test and reference PNGs are validated and decoded using the same capture contract. Decoding preserves the stored samples and does not convert them to sRGB, reduce their bit depth, gamut-map them, or tone-map them. Comparison uses decoded channel values rather than encoded PNG bytes, since different PNG encodings can represent the same samples.

Exact comparison is sufficient for synthetic fixtures and deterministic paths. Existing fuzzy semantics can continue to apply to compatible 8-bit samples. Higher-precision samples need follow-up work to define the comparison domain, tolerance units, alpha handling, and the mapping from existing `maxDifference` and pixel-count values. Until then, higher-precision tests use exact comparison, and JPEG XL tests that require a tolerance remain manual.

`wptrunner` passes the resolved request to the executor and preserves the original PNG for validation and diagnostics. The requested color space is part of the screenshot cache key so captures of the same URL in different color spaces do not collide. A vendor runner may use a different capture path as long as it produces output satisfying the same capture contract and compares the same decoded samples.

On a rendering mismatch or invalid capture, the runner keeps the available original PNGs as artifacts. A derived sRGB or tone-mapped preview may be provided for debugging, but it is not used for comparison.

## Risks

- **Decoder support.** Existing Python image libraries and WPT code paths may reduce precision, convert to sRGB, discard color signaling, or tone-map. The color-preserving path needs targeted fixtures for these failures.
- **Protocol and implementation dependency.** WPT cannot enable a color space until the WebDriver specifications define the corresponding PNG output and at least one browser and runner can produce and decode it. `display-p3` is the intended first end-to-end path; PQ remains dependent on protocol and implementation feasibility.
- **Fuzzy comparison.** Exact comparison is likely too strict for broad JPEG XL coverage, but an underspecified higher-precision tolerance would produce inconsistent results across runners.
- **Reference quality.** A reference containing only sRGB or SDR values makes an opted-in test ineffective.
- **Cost and debugging.** Higher-bit-depth PNGs require more storage and decode time and may not display correctly in ordinary image viewers.
- **Downstream consumers.** Vendor harnesses, dashboards, and artifact systems need to handle the new metadata, capture errors, and image artifacts.

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
