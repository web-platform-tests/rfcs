# RFC 226: Tentative testdriver methods

## Summary
Allow tentative methods to be added to testdriver.js that are not yet included in the WebDriver or WebDriver BiDi specifications.
This should only be allowed where there is consensus between multiple browser vendors and where there is clear progress towards a specification.

## Background
As part of the [Interop 2025 Accessibility Investigation Area](https://github.com/web-platform-tests/interop-accessibility/issues/148), the intention is to extend browsers and WPT to allow additional accessibility properties to be tested by web platform tests.
However, there are open questions regarding the shape of the API for exposing these properties, what properties should be exposed, how the tests should be written, etc.
While the end goal is to extend the WebDriver specification with the required new endpoints, these open questions need to be answered before this is feasible.
To answer these questions, it would be helpful if browser vendors could collaborate on a tentative but working implementation.

## Details
1. There must be publicly documented (e.g. as part of an Interop Investigation Area) consensus between at least two browser vendors and intent to work towards a specification.
2. Any tentative methods added to testdriver.js should be prefixed with `tentative_`.
3. Any utility methods which call tentative testdriver methods should also be prefixed with `tentative_`.
4. Any tests directly or indirectly calling tentative testdriver methods must be marked tentative.

## Alternatives considered
1. Avoid any changes to the WPT repository until the specification is finalised.
  This makes it very difficult for browser vendors to collaborate and to learn about problems that are much easier to discover and understand "in practice".
  In contrast, if the specification were developed without a working implementation, there is a much higher risk of fundamental design problems in the specification which are much harder to fix later.
2. Develop the implementation as vendor specific tests with methods in testdriver-vendor.js.
  While this does allow a single vendor to iterate on a working implementation, it is still very difficult for multiple browser vendors to collaborate on this.
  At the very least, other vendors may wish to contribute tests which exercise the implementation to ensure it fits their needs before agreeing to a final specification.

## Risks
Methods could be added which never end up being specified, resulting in cruft and non-standardised functionality.
This is not a significant risk to the web at large because these methods only impact tests and the tests must be marked tentative, preventing them from being considered for Interop scoring, for example.
