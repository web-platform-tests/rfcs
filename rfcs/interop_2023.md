# RFC 116: Interop 2023

## Summary

Interop 2023 is an effort to improve interoperability in areas believed to be important to web developers and users. This is the next iteration of [Interop 2022](./interop_2022.md), an effort which is still ongoing at the time of this RFC.

The focus areas will be selected by a proposals period followed by a proposals review and selection process. The [Interop team](https://github.com/orgs/web-platform-tests/teams/interop-2022) will manage this process and make decisions based on consensus, see [governance](#governance).

There will be a dashboard at https://wpt.fyi/interop-2023.

## Details

The Interop 2022 [repo](https://github.com/web-platform-tests/interop-2022) and [team](https://github.com/orgs/web-platform-tests/teams/interop-2022) will be renamed to omit the year and used for this effort as well.

### Timeline

During 2022:

- September 15 to October 15: Proposals period.
- October 16 to October 30: Proposal discussion/refinement. (New proposals no longer accepted.)
- November 1 to November 30: Proposal selection. (The team must reach consensus no later than November 30.)
- December 1 to December 16:
  - Start review of tests. Write needed tests.
  - Start involving PR teams. Write rough drafts of announcements. Begin creating dashboard.

During 2023:

- January 9 to January 20:
  - Finalize test selection.
  - Complete announcement plans. Prepare dashboard for launch.
- January 23 to January 27: Launch.

If necessary, additional changes to the test lists can be made throughout the year with the consensus of the team, see [updating the tests](#updating-the-tests).

### Governance

The [Interop team](https://github.com/orgs/web-platform-tests/teams/interop) will manage the overall Interop 2023 effort. The team will be formed by renaming the [’22 team](https://github.com/orgs/web-platform-tests/teams/interop-2022).

The team will meet regularly, with agenda issues created in the repo at least 48 hours in advance, and meeting minutes posted in the issue.

The team makes decisions based on consensus, which is defined as support from at least two participating organizations and no opposition.

The Interop 2023 effort is subject to the [WPT code of conduct](https://github.com/web-platform-tests/wpt/blob/master/CODE_OF_CONDUCT.md).

### Proposals

During the proposals period (September 15 to October 15), proposals are submitted as issues in the [Interop repo](https://github.com/web-platform-tests/interop). A proposal should be as specific as reasonably possible, but multiple proposals may be grouped later in the selection process.

There are two kinds of proposals: focus areas where progress is measured using test results, and investigation efforts where progress is measuring against a task list. For both kinds of proposals, finishing them should meaningfully advance the state of interoperability for the web platform.

#### Focus areas

These proposals are for existing web platform features with good a spec and automated test suite which can be used to improve interoperability. Progress is measured by test results on wpt.fyi.

The following information should be provided in a proposal:

- Is this area a problem for web developers? For example, survey data and browser bug counts can help.
- Is this area a problem for users?
- How widely used is this feature? For example, [use counters](https://www.chromestatus.com/metrics/feature/popularity), [HTTP Archive](https://httparchive.org/) and [State of CSS](https://2021.stateofcss.com/en-US/features/) can help.
- Specification for this feature.
- A list of tests on wpt.fyi which will be used to score progress towards interoperability on the feature.

#### Investigation efforts

If an area doesn't have a good spec and test suite it's not possible to use wpt.fyi test results to judge progress. Instead, some other work is needed to bring the area to state where test results can be used.

The following information should be provided in a proposal:

- Is this area a problem for web developers? For example, survey data and browser bug counts can help.
- Is this area a problem for users?
- How widely used is this feature? For example, [use counters](https://www.chromestatus.com/metrics/feature/popularity), [HTTP Archive](https://httparchive.org/) and [State of CSS](https://2021.stateofcss.com/en-US/features/) can help.
- Specification for this feature, if any.
- A list of tasks which should be completed as part of the effort. This could be producing a report for a working group, a list of spec bugs which should be resolved, or tests/infrastructure which need to be created. A completed investigation effort should unblock the feature for inclusion as a regular focus area in the future.

### Proposal selection

Proposal selection (November 1 to November 30) will be done using a positions process similar to [Interop 2022](https://github.com/web-platform-tests/interop-2022/issues/38) but with some changes. The process is:

- Each participating organization takes one of 3 positions on each proposal: support, neutral, or oppose.
- A proposal is accepted if there is consensus, meaning support from at least two participating organizations and no opposition.
- The team reviews the accepted proposals and can decide to group multiple proposals, or to drop proposals. Both actions require team consensus.
- The final list of accepted proposals is decided by team consensus no later than November 30.

The purpose of the second round of grouping and optionally dropping proposals is to avoid sub-proposals as in the Interop 2022 process. Instead, proposals will be as specific as reasonably possible, and can be grouped into themes afterwards. The expectation is that proposals will only be dropped if there is consensus that they don't make sense in light of which other proposals were accepted.

### Updating the tests

No test suite is perfect, and there will likely be a need to add, remove or rewrite tests during the course of the year. To suggest changes, file an issue on the [Interop repo](https://github.com/web-platform-tests/interop). The team will make decisions based on consensus, see [governance](#governance).

### Metrics

Per-area and overall metrics will be defined in a similar fashion to [Interop 2022](./interop_2022.md#metrics). The number of areas and their contribution to the total score will be decided in the proposal selection phase.

### Dashboard

A public dashboard tracking the metrics over time will be available at https://wpt.fyi/interop-2023.

## Risks

No set of tests is without flaws, and trying to improve a metric based on test results can lead to bad outcomes such as shallow implementations that pass the tests but still isn't usable in practice. The main mitigation for this is a careful review of the tests for the proposed areas, and to include only tests that are judged to be high quality. Also, the tests are not frozen but can be updated or even removed during the year as issues are uncovered.
