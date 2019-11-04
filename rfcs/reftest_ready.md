# RFC 34: `reftests`: Fire event when painting is done

## Summary

Fire a `TestRendered` event for reftests when a `reftest-wait` class is present on the root and layout, fonts and painting are complete. This corresponds to the point at which the screenshot would be taken in the absence of `reftest-wait`.

## Details

Reftests which want to check for behaviour in the face of dynamic changes need to know when those changes can be made with a complete initial state. This corresponds to the time that a screenshot would be taken in the case that the `reftest-wait` attribute is not present. Firing an event at this time allows the test to schedule dynamic changes and then remove the `reftest-wait` attribute to trigger a screenshot without concern that the changes are batched into the intial render.

The detailed steps for running a reftest with this addition are as follows:

* Wait for the document to complete loading, all fonts to load, and for pending paints to be flushed.
* If the document root element does not contain a class named `reftest-wait`, screenshot the viewport and abort these steps
* Fire an event at the document root with the name `TestRendered` and the bubbles attribute set to true.
* Wait for the `reftest-wait` class to be removed from the document root
* Wait for pending paints to flush
* Screenshot the viewport

This addition corresponds to the gecko `MozReftestInvalidate` event. Providing this event is expected to unblock upstreaming some more Gecko reftests.

## Example

Adapted from the [gecko docs](https://searchfox.org/mozilla-central/source/layout/tools/reftest/README.txt#509)

```js

function doTest() {
  document.body.style.border = "";
  document.documentElement.removeAttribute('class');
}
document.addEventListener("TestRendered", doTest, false);
```      

## Risks

No expected risks except those introduced by any feature e.g. complexity.

The event itself may not be enough to test invalidation codepaths in browsers; browser-specific approaches may be required to do that. However it's a prerequisite for that work.
