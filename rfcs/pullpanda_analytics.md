# RFC 35: Add third-party GitHub App: Pull Panda

## Summary

At TPAC 2019 we set out some [priorities for 2020](https://docs.google.com/document/d/1gie7LFb6cAUfabY86MYuWM7m7ux_FaKhDkLdpz0zWkg/edit),
which include increasing PR review velocity and reduce the number of stale PRs.

In order to track progress over time,
install the [Pull Panda](https://pullpanda.com/) GitHub App
for the web-platform-tests/wpt repository (or all repos in the org).

Pull Panda can show [analytics](https://pullpanda.com/analytics) for the past 3 months for, among other things,
PR merge time and open PRs,
which are relevant for tracking the above-mentioned priorities.

No setup required, only give it read permission to the org or repo.
The analytics should be [visible to all organization members](https://docs.pullpanda.com/products/pull-analytics/user-access).
This app is owned by GitHub.

## Details

Pull Panda Analytics has these graphs and explanations:

* Review turnaround
* Reviewer workload: Shows the number of PRs currently assigned to reviewers.
* Open PRs: This is the number of pull requests open right now and at the start of each week in the past.
* PR merge time
  - Breakdown of merge time: Merge time is the time it takes for a pull request to go from opened to merged.
  - Average merge time: This is the average (mean) pull request merge time by week. Merge time is the time it takes for a pull request to go from opened to merged.
* PR throughput
  - PRs merged: This shows the number of pull requests opened and merged by week.
  - PR status by week opened: This is a cohort analysis showing the status of pull requests grouped by the week they were opened.
* PR size
  - PR size is caclulated as the sum of lines added and removed. For example, if a pull request has 5 lines added and 5 lines removed, its size is 10 LOC.
  - Success rate (based on a target of 500 LOC (custimizable)): This shows the percentage of pull requests that met the target. PR size is calculated as the sum of lines added and removed.
* Code churn
  - Additions and deletions: This shows the number of lines added and deleted in merged pull requests.
  - Contributors: This shows the number of users who authored merged pull requests.

Pull Panda can be installed on the whole organization or select repositories.
It only requests read access for analytics.

Pull Panda was [acquired by GitHub](https://pullpanda.com/github) in June, 2019.
It is free to use for GitHub orgs.

## Risks

3 months is not enough history to see trends on a longer time scale.
There is [an open issue](https://github.com/pullreminders/backlog/issues/94) about this.

Trust. This risk is severely mitigated by the granted permission being read-only
and that the app is owned by GitHub.

## Alternatives considered

https://github.com/cncf/devstats supports showing graphs for various interesting things, in particular
PR Time to Approve and Merge ([example](https://k8s.devstats.cncf.io/d/44/pr-time-to-approve-and-merge?orgId=1)),
Open PR Age by Repository Group ([example](https://k8s.devstats.cncf.io/d/25/open-pr-age-by-repository-group?orgId=1)) and
Open issues/PRs by milestone and repository ([example](https://k8s.devstats.cncf.io/d/22/open-issues-prs-by-milestone-and-repository?orgId=1)).

This was pretty involved in its setup.
Initial attempts to configure locally on macOS were unsuccessful.
Either a project can be added to devstats.cncf.io,
or something can be set up with Travis CI,
but these options have not been further investigated as both seem non-trivial.
