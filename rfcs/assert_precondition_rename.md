# RFC 43: Split 'assert\_precondition' into optional and non-optional variants

## Summary

Split the `assert_precondition` function into `assert_implements_optional` and
`assert_implements`. The `assert_implements_optional` function will behave as
`assert_precondition` does today; a failed assert will record a
'PRECONDITION\_FAILED' status. The `assert_implements` function will be
synactical sugar for `assert_true`.

Reuse the 'PRECONDITION\_FAILED' status rather than renaming it due to the
technical difficulties of getting mozlog updated.

## Details

### Background 

The `assert_precondition` function and associated status
('PRECONDITION\_FAILED') were added in [RFC #16](assert_precondition.md).
However there has been confusion over whether the function is meant for spec
features that are explicitly marked OPTIONAL (i.e. its usage in
`html/semantics/embedded-content/the-video-element/resize-during-playback.html`),
or whether it is meant as an early-out when a particular spec or feature is not
implemented by a browser (i.e. its usage in `portals/`).

The difference matters here due to the 'PRECONDITION\_FAILED' status, which has
been interpreted by some users as a 'non-failing' status - which only makes
sense if the failure is for truly OPTIONAL behavior.

### Proposal

Rename the `assert_precondition` function to `assert_implements_optional`, with
the same 'PRECONDITION\_FAILED' status produced, and add a new function`
assert_implements` that is essentially syntactical sugar for `assert_true`.
That is:

> __assert_implements(condition, description)__
>
> Concludes the test with a `FAIL` status if `condition` is not truthy.
> Used to avoid running unnecessary code or subtests for a feature that is not
> implemented by the running browser.

> __assert_implements_optional(condition, description)__
>
> Concludes the test with a `PRECONDITION_FAILED` status if `condition` is not truthy.
> Used for skipping tests or subtests for OPTIONAL features that that would otherwise
> fail due to lack of support for the feature.

The 'PRECONDITION\_FAILED' status is not being renamed, as changing the logging
infrastructure (mozlog, etc) is logistically difficult.

## Risks

* As with any change in testharness.js, downstream embedders may have to spend
  effort to adapt to the deprecation and rename. That said, there are currently
  zero uses of `assert_precondition` in downstream Chromium tests.
* As it is not being renamed, the 'PRECONDITION\_FAILED' status may confuse
  consumers of WPT results.
