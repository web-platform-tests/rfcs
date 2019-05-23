# Load Ahem as a Webfont always

## Summary
Make all tests load the Ahem font as a webfont instead of expecting Ahem to be
installed as a system font.

## Details
Ahem is used in a range of reftests because it has glyphs of a very precise size
and shape. WPT currently assumes that Ahem is installed as a system font, and
provides a mechanism to install fonts where not available.

However, some environments (such as CI systems) may not have this font installed
and it may not be feasible to install it (eg: on Chromium CI).

The proposal is to change this assumption and require all tests to load Ahem as
a webfont. Here is a proposed PR for doing this and updating all existing tests:
https://github.com/web-platform-tests/wpt/pull/16951

## Risks
The main risk is inability to test behaviour specific to having this font as a
system font, although no such tests seem to exist today.

Another is that test runtime may be slower due to the network latency of loading
the font, but this is expected to be small (if detectable at all) due to
caching.
