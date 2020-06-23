# RFC 58: Add a new test property to control displaying test state updates

## Summary

Add a new test property called `hide_test_state` to control displaying
test state updates while tests are running.

This property will default to `false`, and can be set to `true` via the `setup()` function.

## Details

In testharness there are a few usage of `show_status()` function,
which outputs the test state to the documents during test
runs. Some tests are sensitive to the content of the documents,
so the test state output may interfere the test results.

We are adding a new setup property to prevent outputting the
test state.

## Risks

No risks as this is an opt-in property.
