# RFC: Codify the `requestAnimationFrame` delay in `test-wait.js` as the test-completion timing

## Summary

Codify the double `requestAnimationFrame` call in wptrunner's `test-wait.js` as the test-completion timing,
updating the documentation to match the implementation.

This delay was added in 2017 as a short-term workaround for a Chromium rendering bug,
but the discrepancy between the documented and actual behaviour has caused recurring flakiness in test runners that implement the documented (no-delay) semantics.

## Background

### Documentation

The [reftest documentation][reftests-docs] describes the screenshot timing as follows:

> By default, reftest screenshots are taken after the following
> conditions are met:
>
> * The `load` event has fired
> * Web fonts (if any) are loaded
> * Pending paints have completed
>
> In some cases it is necessary to delay the screenshot later than this,
> for example because some DOM manipulation is required to set up the
> desired test conditions. To enable this, the test may have a
> `class="reftest-wait"` attribute specified on the root element. In
> this case the harness will run the following sequence of steps:
>
> * Wait for the `load` event to fire and fonts to load.
> * Wait for pending paints to complete.
> * Fire an event named `TestRendered` at the root element, with the
>   `bubbles` attribute set to true.
> * Wait for the `reftest-wait` class to be removed from the root
>   element.
> * Wait for pending paints to complete.
> * Screenshot the viewport.

The [crashtest documentation][crashtests-docs] describes similar but distinct behaviour:

> The simplest crashtest is a single HTML file with any content. The
> test passes if the load event is reached, and the browser finishes
> painting, without terminating.
>
> In some cases crashtests may need to perform work after the initial page load.
> In this case the test may specify a `class=test-wait` attribute on the root
> element. The test will not complete until that attribute is removed from the
> root. At the time when the test would otherwise have ended a `TestRendered`
> event is emitted; test authors can use this event to perform modifications that
> are guaranteed not to be batched with the initial paint. This matches the
> behaviour of reftests.

Despite the implementation in most wptrunner executors being identical,
the crashtest documentation does not mention web fonts or pending paints,
only stating that "this matches the behaviour of reftests":
it's unclear whether that's meant to be an informative statement or a definition-by-reference.

### wptrunner

The wptrunner implementation in [`test-wait.js`][test-wait-js],
which is used for reftests (except in Firefox, see below) and crashtests,
waits for two animation frames using a nested pair of `requestAnimationFrame` callbacks.
This wait runs after font loading and again after the `reftest-wait` class is removed,
so it is the actual implementation of *both* the documented "pending paints" steps.
This was [added in April 2017][wpt-bea0e9] ([wptrunner PR][wptrunner-248]) as a workaround for [Chromium bug 708757][cr-bug] (now Chromium issue 41311503),
where screenshots were taken before images were fully rendered despite `document.readyState` being `"complete"`.
This is described by a comment as "a short-term workaround", but is now nine years old.

The "pending paints" clauses were written later
(as part of the [`TestRendered` event RFC][test-rendered-rfc] in 2019)
without acknowledging the existing `requestAnimationFrame` delay,
which is in practice the only mechanism wptrunner uses to wait for paints.

### WebDriver

Both [WebDriver Classic][webdriver-classic] and [WebDriver BiDi][webdriver-bidi] synchronize screenshots
to the HTML spec's [run the animation frame callbacks][run-rAF] step.
Neither spec defines a mechanism for waiting for pending paints to complete,
though the BiDi spec has [an inline issue][webdriver-bidi-issue-b2b83ca0] about the shortcomings of this definition:
"This ought to be integrated into the update rendering algorithm in some more explicit way".

Since wptrunner uses WebDriver to control most browsers
(except Firefox, which has its own internal reftest runner),
the double-`requestAnimationFrame` delay in `test-wait.js` is the only mechanism provided
to ensure rendering has settled before the screenshot.

### Chromium bug

The status of the [Chromium bug][cr-bug] that was originally being worked around is unclear.

In 2017, sc...@chromium.org [believed all image loading event timing issues][cr-comment8] had been addressed in [another bug][cr-access-denied]
("Access is denied to this issue", so no further context is available).

In a [2019 discussion][cr-comment21],
lp...@chromium.org confirmed that the double-`requestAnimationFrame` workaround in WPT was the reason Chromium did not see test flakiness,
noting that neither WebDriver nor Puppeteer's screenshot APIs attempt to ensure painting has completed—they simply capture whenever requested.

Despite this, in 2020 it was [marked as fixed][cr-comment25] on the basis of the fixes in the other bug in 2017.

However, in 2023,
Chromium [adopted the same double-`requestAnimationFrame` pattern][cr-2ef098] in their own `content_shell` test runner,
explicitly to match the wptrunner behaviour,
noting that they were still seeing cases where tests ended prematurely under `run_web_tests.py`.

It is unclear whether the cause of that 2023 flakiness was ever investigated.

### Firefox reftest behaviour

Firefox runs reftests through its own internal runner
rather than `test-wait.js` and WebDriver.

It relies on `MozAfterPaint` firing,
which I believe guarantees at least one [update the rendering][html-update-the-rendering] step has run,
but does not guarantee that it has run twice and two `requestAnimationFrame` events have fired.

### WebKit reftest behaviour

For Safari and Safari Technology Preview runs on wpt.fyi,
see the wptrunner and WebDriver sections above.

When running WPT within WebKit via `run-webkit-tests`,
before taking a screenshot, both `DumpRenderTree` and `WebKitTestRunner` perform a synchronous paint flush without yielding to the event loop at all,
which means `requestAnimationFrame` has no opportunity to fire.

## Problem

The discrepancy between the documented and actual test completion timing causes test flakiness.

`run-webkit-tests` flushes pending paints synchronously before the snapshot
but does not perform a double-`requestAnimationFrame` delay,
and this has required a variety of test fixes:

* Adding `reftest-wait` with an explicit `document.fonts.ready` and double-`requestAnimationFrame`
  due to the web font triggering re-layout after it loaded,
  and thus merely waiting for paint isn't enough
  ([WebKit commit `b06fedead73a`][webkit-b06fed];
  see also [csswg-drafts issue #4248][csswg-4248]).
* Adding the missing `class="reftest-wait"` to a scroll-position reftest
  whose inline `requestAnimationFrame` scroll callbacks were skipped by WebKit's synchronous flush,
  a bug masked by wptrunner's implicit delay
  ([WebKit commit `136c73f1ef50`][webkit-136c73]).
* Adding `reftest-wait` to a reftest that calls `.focus()` at load time,
  deferring the screenshot until the focus style had been rendered
  ([WebKit commit `300a05321aff`][webkit-300a05]).

In each case, forcing a paint synchronously at the time of the screenshot was insufficient.

## Details

Codify the double-`requestAnimationFrame` delay as the official test-completion timing:

1. Update the [reftest documentation][reftests-docs] to replace each "pending paints" step
   with the two `requestAnimationFrame` callbacks that are its actual implementation.

2. Update the [crashtest documentation][crashtests-docs] to match,
   with `test-wait` in place of `reftest-wait`
   and test completion in place of screenshotting.

3. Remove the "short-term workaround" comment from `test-wait.js`.

This makes the documented behaviour match the actual behaviour of `test-wait.js`.

It does, therefore, require `run-webkit-tests` and Firefox's internal reftest runner to change their behaviour.

## Risks

### Slower test execution

Every reftest and crashtest pays the cost of two animation frame delays.
This is already the case in wptrunner and Chromium's `content_shell` runner,
but test runners that currently do not implement the delay would need to add it.

### Not a robust guarantee
The [2019 description of Chromium's behaviour][cr-comment21] states that WebDriver and Puppeteer just take a screenshot when requested.
Codifying this behaviour does not guarantee that all rendering will have settled within two animation frames,
though that is an assumption made within various places within web-platform-tests.

### Codifying creates inertia

If a more robust mechanism for waiting for pending paints becomes available in WebDriver,
codifying the double-`requestAnimationFrame` delay as official behaviour makes it harder to change.

## Alternatives considered

### Restrict the double-`requestAnimationFrame` to Chromium

We could limit the behaviour to Chromium, however:

* We would continue to see the flakiness in other browsers,
  for example with the `fonts.ready` resolve-time ambiguity.
* It wouldn't provide Chromium contributors any more incentive to fix their tests cross-browser.

### Remove the double-`requestAnimationFrame`

Instead of codifying the behaviour,
we could remove them from `test-wait.js`.

However:

* It may re-expose Chromium to their earlier test flakiness,
  if it hasn't been fully addressed.
* We would continue to see the flakiness in other browsers,
  for example with the `fonts.ready` resolve-time ambiguity.
* Tests that currently pass under wptrunner would become flaky if they depend on the delay.
  While these could be treated as test bugs and fixed with explicit `reftest-wait`/`test-wait` synchronization,
  this would likely require significant effort across the test suite,
  especially when we have limited tools to observe flakiness.

### Implement a proper "pending paints" mechanism

Rather than codifying the `requestAnimationFrame` workaround,
wptrunner could implement a more robust mechanism for waiting for pending paints to complete.
This would address the inertia concern above,
but no such mechanism currently exists or has been proposed.

[reftests-docs]: https://web-platform-tests.org/writing-tests/reftests.html
[crashtests-docs]: https://web-platform-tests.org/writing-tests/crashtest.html
[test-wait-js]: https://github.com/web-platform-tests/wpt/blob/master/tools/wptrunner/wptrunner/executors/test-wait.js
[cr-bug]: https://issues.chromium.org/issues/41311503
[cr-access-denied]: https://issues.chromium.org/issues/40079697
[cr-comment8]: https://issues.chromium.org/issues/41311503#comment8
[cr-comment21]: https://issues.chromium.org/issues/41311503#comment21
[cr-comment25]: https://issues.chromium.org/issues/41311503#comment25
[cr-2ef098]: https://chromium.googlesource.com/chromium/src/+/2ef098fcae2c61d89dda0fdfd6979a0a1afcec63
[webkit-b06fed]: https://github.com/WebKit/WebKit/commit/b06fedead73a
[webkit-136c73]: https://github.com/WebKit/WebKit/commit/136c73f1ef50
[webkit-300a05]: https://github.com/WebKit/WebKit/commit/300a05321aff
[wpt-bea0e9]: https://github.com/web-platform-tests/wpt/commit/bea0e994482
[wptrunner-248]: https://github.com/w3c/wptrunner/pull/248
[test-rendered-rfc]: https://github.com/web-platform-tests/rfcs/blob/master/rfcs/test_rendered_event.md
[webdriver-classic]: https://w3c.github.io/webdriver/#take-screenshot
[webdriver-bidi]: https://w3c.github.io/webdriver-bidi/#command-browsingContext-captureScreenshot
[webdriver-bidi-issue-b2b83ca0]: https://w3c.github.io/webdriver-bidi/#issue-b2b83ca0
[run-rAF]: https://html.spec.whatwg.org/multipage/imagebitmap-and-animations.html#run-the-animation-frame-callbacks
[html-update-the-rendering]: https://html.spec.whatwg.org/multipage/webappapis.html#update-the-rendering
[csswg-4248]: https://github.com/w3c/csswg-drafts/issues/4248
