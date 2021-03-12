# RFC 76: wptpr.live turndown

## Summary

Turn down http://wptpr.live, which allows project contributors to preview their
submissions during the peer review process, due to lack of uptake/interest and
overhead of maintaining the system.

This RFC does not affect http://wpt.live, which will continue to run.

## Details

In [RFC 36](https://github.com/web-platform-tests/rfcs/pull/36), which landed
in Nov 2019, Bocoup and Google's Ecosystem Infra team jointly proposed bringing
up both a public hosted copy of the WPT repository (http://wpt.live) and a
PR-preview service (http://wptpr.live). The goal was to support reviewers of
test changes in WPT, by letting them preview the changes without needing to run
WPT locally.

Unfortunately however the system has faced a number of technical issues
(historical and ongoing), as well as limited uptake from users (not least
because it is difficult to locate in the GitHub PR workflow). Bocoup is also no
longer involved in maintaining the project. Given this situation, we either
need to invest heavily in the project (to reduce its long-term maintenance cost
and improve the PR workflow integration), or make the decision to shut it down.

Due to current and expected available engineer resourcing, this RFC proposes
shutting down http://wptpr.live. This will remove the ability to preview
in-flight pull requests in WPT without checking out the branch in a local clone
of WPT. As noted in the Summary, this will not affect http://wpt.live.

## Risks

The obvious risk here is an impact to the efficacy of PR reviews, if reviewers
are relying on http://wptpr.live. Unfortunately the page does not have
analytics, so the actual user-count is unknown, but anecdotally few users have
mentioned the desire to use this service over the year it has been available.
Moreso, http://wptpr.live didn't work for pull requests from forks from Feb
2020 until Dec 2020, and again anecdotally almost nobody noticed.
