# RFC 99: Interop 2022

## Summary

Identify focus areas and a list of tests for Interop 2022, a public effort and benchmark around which browser vendors can collaborate to improve browser compatibility in areas believed to be important to web developers.

Proposed focus areas are tracked in the dedicated [Interop 2022 repo](https://github.com/web-platform-tests/interop-2022).

## Details

10 new focus areas are part of Interop 2022, as well as 5 from Interop 2022.

### Cascade Layers

Proposal: https://github.com/web-platform-tests/interop-2022/issues/5
Tests: [`interop-2022-cascade`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2022-cascade)

### Color Spaces & CSS Color Functions

Proposal: https://github.com/web-platform-tests/interop-2022/issues/20

Tests: [`interop-2022-color`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2022-color)

### Containment

Proposal: https://github.com/web-platform-tests/interop-2022/issues/19

Tests: [`interop-2022-contain`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2022-contain)


### `<dialog>` and `::backdrop`

Proposal: https://github.com/web-platform-tests/interop-2022/issues/12

Tests: [`interop-2022-dialog`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2022-dialog)

### Form Controls

Proposal: https://github.com/web-platform-tests/interop-2022/issues/11

Tests: [`interop-2022-forms`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2022-forms)


### Scrolling

Proposals: https://github.com/web-platform-tests/interop-2022/issues/14 + https://github.com/web-platform-tests/interop-2022/issues/21

Tests: [`interop-2022-scrolling`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2022-scrolling)

### Subgrid

Proposal: https://github.com/web-platform-tests/interop-2022/issues/1

Tests: [`interop-2022-subgrid`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2022-subgrid)

### Text

Proposal: https://github.com/web-platform-tests/interop-2022/issues/13

Tests: [`interop-2022-text`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2022-text)

### New Viewport Units

Proposal: https://github.com/web-platform-tests/interop-2022/issues/4

Tests: `interop-2022-viewport` (TODO)

See also the [Viewport Investigation project](https://github.com/web-platform-tests/interop-2022/issues/41) which is part of the Interop 2022 effort, but not part of the metric.

### Web Compat

Proposal: https://github.com/web-platform-tests/interop-2022/issues/9

Tests: [`interop-2022-webcompat`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2022-webcompat)

### Interop 2021

The 5 focus areas from Interop 2021 are carried forward and included in the metric. See https://github.com/web-platform-tests/interop-2022/issues/2 for discussion.

- [`interop-2021-aspect-ratio`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2021-aspect-ratio)
- [`interop-2021-flexbox`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2021-flexbox)
- [`interop-2021-grid`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2021-grid)
- [`interop-2021-position-sticky`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2021-position-sticky)
- [`interop-2021-transforms`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2021-transforms)

[Query for `interop-2021-*`](https://wpt.fyi/results/?label=master&label=experimental&product=chrome&product=firefox&product=safari&aligned&q=label%3Ainterop-2021-aspect-ratio%20or%20label%3Ainterop-2021-flexbox%20or%20label%3Ainterop-2021-grid%20or%20label%3Ainterop-2021-position-sticky%20or%20label%3Ainterop-2021-transforms)

### Metrics

Scores (between 0% and 100%) will be computed from test suite pass rates for each focus area and the effort as a whole. In total there are 15 focus areas, of which 5 are carried forward from Interop 2021. All areas are given equal weight, meaning that the new areas make up 2/3 of the score and the old ones 1/3 of the score.

Scoring in detail:

- Every test is scored between 0 and 1. For tests with subtests, the score is the number of passes divided by the number of subtests. This avoids tests with many subtests being given much larger weight than for example reftests.
- Every area is scored between 0 and 1, adding up the scores and dividing by the number of tests.
- The overall score is the sum of area scores divided by the number of areas (15).

Scores are presented as percentages.

TODO: decide on floating point vs. integer vs. rational numbers

### Dashboard

A public dashboard tracking the metrics over time will be available at https://wpt.fyi/interop-2022. The dashboard will include at least:

- An explanation of the scope, which is the 15 focus areas, not the whole web platform.
- A graph of the metric over time for at least Chrome, Firefox and Safari, which can show both the overall Interop 2022 metric, the Interop 2021 metric which makes up 1/3 of the score, and each of the 15 focus areas individually.
- A brief description of each focus area, with links to specs, tests on wpt.fyi, and documentation on MDN.

Additional features might be added, see https://github.com/web-platform-tests/interop-2022/issues/45 for discussion.

### How to nominate an area

_The time to nominate areas has already passed._

To propose an area for inclusion Interop 2022, comment on the [pull request](https://github.com/web-platform-tests/rfcs/pull/99) and provide as much as possible of the following information:

- How widely used is this feature? For example, [use counters](https://www.chromestatus.com/metrics/feature/popularity), [HTTP Archive](https://httparchive.org/) and [State of CSS](https://2020.stateofcss.com/en-US/features/) can help.
- Is this area a problem for web developers? For example, survey data and browser bug counts can help.
- Current test results on wpt.fyi

## Background

Following [MDN Web DNA 2019](https://insights.developer.mozilla.org/reports/mdn-web-developer-needs-assessment-2019.html), the [browser compat report](https://insights.developer.mozilla.org/reports/mdn-browser-compatibility-report-2020.html) narrowed in on a small number of concrete areas where web developers struggle with browser compatibility. Based on that and [other signals](https://web.dev/compat2021/#choosing-what-to-focus-on), [Compat 2021](https://wpt.fyi/compat2021) was created. Over the course of 2021, that effort has improved test results in the focus areas considerably, and the [mid-year](https://web.dev/compat2021-midyear/) and [holiday updates](https://web.dev/compat2021-holiday-update/) outline what this means in practice for web developers.

While Compat 2021 was discussed among all major browser vendors, it didn't follow the web-platform-tests RFC process, and only Google and Microsoft have publicly committed to the effort. Interop 2022 is proposed the successor to Compat 2021, but following a public process of nomination and review, using the RFC process. The term "interop" is used instead of "compat" to avoid conflation with "site compat", see [terminology](#terminology) for details.

## Risks

### Incentives

No set of tests is without flaws, and trying to improve a metric based on test results can lead to bad outcomes such as shallow implementations that pass the tests but still isn't usable in practice. The main mitigation for this is a careful review of the tests for the proposed areas, and to include only tests that are judged to be high quality. Also, the tests are not frozen but can be updated or even removed during the year as issues are uncovered.

### Terminology

The terms "compatibility" and "interoperability" are typically distinguished by browser vendors, where compat refers to site compat, and interop refers to two or more browsers behaving the same. In that terminology, the web-platform-tests project and this effort is about interoperability.

The word "compatibility" is probably more familiar to web developers, and is the terminology of MDN's [browser compatibility tables](https://developer.mozilla.org/en-US/docs/Web/API/AudioTrack#browser_compatibility).

This effort uses the accepted terminology among browser vendors, and is thus named Interop 2022 instead of Compat 2022.