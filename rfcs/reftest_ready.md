# RFC 34: `reftests`: Fire event when painting is done

## Summary

Fire a `ReftestReady` event when layout, fonts and paiting are complete. This coresponds to the point at which the screenshot will be taken in the absence of `reftest-wait`.

## Details

Reftests which want to check for behaviour in the face of dynamic changes need to know when those changes can be made with a complete initial state. This corresponds to the time that a screenshot would be taken in the case that the `reftest-wait` attribute is not present. Firing an event at this time allows the test to schedule dynamic changes and then remove the `reftest-wait` attribute to triger a screenshot without concern that the changes are batched into the intial render. For want of a better name the event fired will be called `ReftestReady`. This corresponds to the gecko `MozReftestInvalidate event. Providing this event is expected to unblock upstreaming some more Gecko reftests.

## Risks

No expected risks except those introduced by any feature e.g. complexity.

The event itself may not be enough to test invalidation codepaths in browsers; browser-specific approaches may be required to do that. However it's a prerequisite for that work.
