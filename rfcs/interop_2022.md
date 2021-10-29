# RFC 99: Interop 2022

## Summary

Identify focus areas and a list of tests for Interop 2022, a public effort and benchmark around which browser vendors can collaborate to improve browser compatibility in areas believed to be important to web developers. Proposed focus areas are tracked in the dedicated [Interop 2022 repo](https://github.com/web-platform-tests/interop-2022).

**Note: This RFC is initially a call for proposals and will be updated include the specific areas before being considered ready for final review.**

## Details

Following [MDN Web DNA 2019](https://insights.developer.mozilla.org/reports/mdn-web-developer-needs-assessment-2019.html), the [browser compat report](https://insights.developer.mozilla.org/reports/mdn-browser-compatibility-report-2020.html) narrowed in on a small number of concrete areas where web developers struggle with browser compatibility. Based on that and [other signals](https://web.dev/compat2021/#choosing-what-to-focus-on), [Compat 2021](https://wpt.fyi/compat2021) was created. Over the course of 2021, that effort has improved test results in the focus areas considerably, and the [mid-year update](https://web.dev/compat2021-midyear/) outlines what this means in practice for web developers.

While Compat 2021 was discussed among all major browser vendors, it didn't follow the web-platform-tests RFC process, and only Google and Microsoft have publicly committed to the effort. Interop 2022 is proposed the successor to Compat 2021, but following a public process of nomination and review, using the RFC process. The term "interop" is used instead of "compat" to avoid conflation with "site compat", see [terminology](#terminology) for details.

No organization is required to make any public commitments.

### Timeline

The proposed timeline is:

- Oct 7 – Nov 7: Anyone can propose an area as an [issue in the Interop 2022 repo](https://github.com/web-platform-tests/interop-2022/issues), ideally with signals of its importance and a proposed list of tests.
- Nov 8 – Nov 21: Browser vendors and others interested in contributing to the effort review the areas and suggest changes to scope, tests, etc. Ranking the proposals by internal priority could also be helpful.
- Nov 22 – Nov 28: Interested parties and the [WPT core team](https://github.com/orgs/web-platform-tests/teams/wpt-core-team/) meet to discuss, seeking to identify 3-7 areas that everyone would be happy to include.
- Nov 29: This RFC is updated and a final call for comments is made.
- Dec 6 – Dec 19: Depending feedback, the RFC is either accepted or rejected.

### How to nominate an area

To propose an area for inclusion Interop 2022, comment on the [pull request](https://github.com/web-platform-tests/rfcs/pull/99) and provide as much as possible of the following information:

- How widely used is this feature? For example, [use counters](https://www.chromestatus.com/metrics/feature/popularity), [HTTP Archive](https://httparchive.org/) and [State of CSS](https://2020.stateofcss.com/en-US/features/) can help.
- Is this area a problem for web developers? For example, survey data and browser bug counts can help.
- Current test results on wpt.fyi

## Risks

### Incentives

No set of tests is without flaws, and trying to improve a metric based on test results can lead to bad outcomes such as shallow implementations that pass the tests but still isn't usable in practice. The main mitigation for this is a careful review of the tests for the proposed areas, and to include only tests that are judged to be high quality. Also, the tests are not frozen but can be updated or even removed during the year as issues are uncovered.

### Terminology

The terms "compatibility" and "interoperability" are typically distinguished by browser vendors, where compat refers to site compat, and interop refers to two or more browsers behaving the same. In that terminology, the web-platform-tests project and this effort is about interoperability.

The word "compatibility" is probably more familiar to web developers, and is the terminology of MDN's [browser compatibility tables](https://developer.mozilla.org/en-US/docs/Web/API/AudioTrack#browser_compatibility).

This effort uses the accepted terminology among browser vendors, and is thus named Interop 2022 instead of Compat 2022.
