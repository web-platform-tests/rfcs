# RFC 119: Allow wptrunner to skip tests that run as expected

## Summary

Add a `--no-repeat-expected` flag to wptrunner to skip tests in subsequent
repeat iterations once they run as expected.

## Details

### Rationale

This flag is used with `--repeat` to implement a retry mechanism.
Retries are intended for CI systems that should be robust to false positives
that unnecessarily block developers:
* Transient infrastructure issues (e.g., CPU utilization spike causing
  timeouts).
* Incomplete expectation data checked into the trunk of a browser vendor's
  source control system.
  The CI check might not surface statuses that should have been added.

### Proposal

When passed `--no-repeat-expected`, wptrunner will run all loaded tests in the
initial repeat iteration.
On subsequent iterations, tests that ran as expected in a previous iteration
emit a `SKIP` status instead of running.
A test ran as expected if that test and its subtests exhibited expected statuses
on all reruns.
The repeat loop can halt early if there are no tests to run.

Restarting between iterations prevents possibly bad state from carrying over and
causing the retry to fail.

### Alternatives

* Using `--repeat` alone and only failing the CI check when a test runs
  unexpectedly consistently.
  Given that almost all tests should run as expected the first time, this scheme
  unnecessarily burdens the CI infrastructure.
* A `--max-retries=<N>` option that retries tests after the repeat loop,
  defaulting to zero.
  This mixes regular repeats with retries in the same wptrunner invocation and
  appears confusingly similar to `--repeat` and `--rerun`.
* Implement retries in vendor CI systems.
  Browser vendors will not benefit from a shared implementation native to
  wptrunner.
  This also adds complexity downstream for merging test results from multiple
  runs.

## Risks

Minimal, since this is an opt-in flag and the default `--repeat` behavior will
not change.

## Appendix

[Prototype](https://github.com/web-platform-tests/wpt/compare/master...jonathan-j-lee:wpt:no-repeat-expected)
