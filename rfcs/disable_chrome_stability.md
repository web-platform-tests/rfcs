# RFC 131: Disable wpt-chrome-dev-stability runs

## Summary

The wpt-chrome-dev-stability job on WPT fails quite frequently and it's
impacting developer productivity. The job should be changed to non-blocking
since it is not providing enough value, while keeping a record of potential
flakiness introductions that could be useful in further investigations.

## Details

Stability jobs in principle help us ensure that new and modified tests aren't
flaky, wasting engineering time down the road as they block landing unrelated
PRs. We currently run stability jobs for Firefox and Chrome, but not when
[merging export PRs from their own repositories](https://github.com/web-platform-tests/wpt/issues/29737).
The reason is that browser engineers usually have better tools or processes in their
internal CI to deal with flaky tests, so if they were to introduce flakiness
for their browser runs, it would be caught and fixed downstream.

A similar argument can be made for stability checks in the other direction. A
Firefox engineer introducing a flaky test that causes wpt-chrome-dev-stability
to fail intermittently would be annoying, but fortunately the Chromium CI has
more tools available for such failures to be properly triaged and investigated.
Therefore wpt-chrome-dev-stability runs are not providing significant value
compared to similar checks in Chromium infra, and are instead harming developer
productivity by blocking the timely landing of unrelated PRs, often from other
browser vendors (e.g. [1](https://github.com/web-platform-tests/wpt/pull/38046#event-8332522292),
[2](https://github.com/web-platform-tests/wpt/pull/38034)).

Disabling the job entirely would also avoid the impact on developer productivity,
which can be considered in the future.

## Risks

New intermittent failures specific to Chromium could be introduced upstream and
only noticed when a WPT import to Chromium CI happens. We believe the tradeoff
is worthwhile, as the Chromium project has robust tooling to investigate and
handle such cases.
