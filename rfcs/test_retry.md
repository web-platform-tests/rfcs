# RFC ?: wptrunner retries tests with unexpected results

## Summary

Add a `--max-retries=<N>` option to wptrunner to retry each test with
unexpected initial results up to `N` times. Once a test runs as expected,
it is not run in subsequent iterations.

The default value is zero, which makes retries effectively opt-in.

## Details

### Rationale

This retry mechanism is intended for CI systems that should be robust to false
positives:
* Transient infrastructure issues (e.g., CPU utilization spike causing timeouts)
* Incomplete expectation data checked into the repo's trunk.
  This can occur when the check gating changes to trunk does not surface the
  missing expected status.
  This can block other developers from making changes while the expectations are
  being corrected.

### Logging

Each retry iteration is preceded by a restart and `suite_start` event and
followed by `suite_end`. The restart prevents possibly bad state from carrying
over.

The (sub)test results themselves are logged as usual through the
`test_{start,end,status}` events.

### Alternatives

* Using `--repeat` and only blocking a change when a test runs unexpectedly
  consistently.
  (Note: This is more lenient than the stability check run in the GitHub repo.)
  This adds a substantial CI burden with little added value, as almost all tests
  run as expected the first time.
* Implement retries in vendor CI systems.
  Browser vendors will not benefit from a shared implementation native to
  wptrunner.
  This also adds complexity downstream for merging test results from multiple
  runs.

## Risks

* Log consumers are incompatible with a variable number of test runs, even
  though each format supports this in principle [TODO: verify].
* Confusion with the similarly named `--repeat` and `--rerun`.
