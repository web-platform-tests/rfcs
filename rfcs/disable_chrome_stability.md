# RFC 131: Disable wpt-chrome-dev-stability runs

## Summary

The wpt-chrome-dev-stability job on WPT fails quite frequently and
it's impacting developer productivity. The job should be changed to
non-blocking, while keeping a record of potential flakiness
introductions that could be useful in further investigations.

## Details

Stability jobs are intended to help us ensure that new and modified
tests aren't flaky. In particular it's designed to ensure that the
test author is aware when they have written a test that is flaky in a
different browser. This is because authors writing a test in a
specific browser engine might be unaware that they are depending on
details of that implementation that make the test unreliable in other
engines. Leaving this to other developers to fix is expensive, because
they have to spend time becoming familiar with details of the test
which are already understood by the original author. In addition, when
there are many unstable web-platfrom-tests, it undermines confidence
in the overall suite and, where such instability is automatically
handled in CI systems (e.g. by marking a test as [PASS, FAIL] in
metadata), reduces the ability of the tests to catch regressions.

Currently, stability checks for the corresponding browser are skipped
when [merging export PRs from their own
repositories](https://github.com/web-platform-tests/wpt/issues/29737)
(i.e. Chromium exports don't run Chrome stability checks, and
likewise for Firefox). This is because we assume that any real
stability issue is likely to have been handled in the source
repository. It's common for such changesets to include code changes
to the browser itself that fix intermittents, and in the source
repository the tests will run together with the corresponding browser
changes. However on GitHub we are using the latest development
release of the browser, which is unlikely to contain code fixes at
the time of export. This makes the stability checks on these PRs
highly prone to misleading failures.

The main problem with the stability checks as currently implemented is
that although the test author is in the best place to understand the
behaviour of the test, they may not understand how to reproduce the
failure in another browser, or may be unable to fix the problem
(e.g. if it's a browser bug rather than a test bug).

Faced with these tradeoffs, the Chromium developers believe the
project is better served by allowing tests which are unstable in
Chrome to land in web-platform-tests and using the tooling they have
available in the Chromium CI system to investigate the
flakiness. Therefore the job will be marked as non-blocking, so we are
able to see where it fails and assess whether the correct tradeoff has
been made.

Marking a job as non-blocking will require changes in the taskcluster
configuration so that the job does not block the sink task.

Disabling the job entirely would also reduce the CI load. This can be
considered once the impact of the change is well understood.

## Risks

New intermittent failures specific to Chromium could be introduced
upstream and only noticed when a WPT import to Chromium CI
happens. Chromium developers believe the tradeoff is worthwhile, as
the Chromium project has robust tooling to investigate and handle such
cases.
