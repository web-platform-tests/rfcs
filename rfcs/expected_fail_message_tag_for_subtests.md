# RFC 165: Support expected-fail-message in test metadata for subtests

## Summary

Add "expected-fail-message" in test metadata so that we can track how
a subtest fails.

## Details

In Chromium developers are using [baselines](https://source.chromium.org/chromium/chromium/src/+/main:third_party/blink/web_tests/external/wpt/editing/event-expected.txt) to track expected output message
for testharness tests. While the output message for a pass subtest is
generally not interesting, the output message for a failed subtest tracks
how a subtest fails, and a change in how a test fails could be unintended
and suggestive of a problem. Chromium developers want to keep such capability
if we want to use Wptrunner to run WPTs in Chromium.

The suggestion is to support "expected-fail-message" tag for subtests. When
present, it takes effect on expected FAIL or expected PRECONDITION_FAILED
results. The test runner should check if the actual output message matches
with the "expected-fail-message". When a mismatch is found, the test runner
should turn the results to unexpected FAIL or unexpected PRECONDITION_FAILED
respectively, otherwise the results continue to be the expected one.

Wdspec tests can also have subtests. "expected-fail-message" should work the
same way for an expected FAIL Wdspec subtest. Other than that, "expected-fail-message"
should not have impact on any other scenarios.

One example metadata file will look like below:
```
[event.html]
  expected: OK
  [Changing selection from handler: input event]
    expected: FAIL

  [Command createLink, value "": input event]
    expected: [FAIL, PRECONDITION_FAILED, PASS]
    expected-fail-message: "number of input events fired expectei 1 but got 0"
```

### Implementations

The current implementation for test metadata parser is flexible enough to
understand any additional tag. The changes needed is 1) wpttest.Test to have one
additional member function to return "expected-fail-message",
2)testrunner.TestRunnerManager.test_ended to have additional logic to turn
expected FAIL or PRECONDITION_FAILED to unexpected failures per the rule
discribed above.

How to add "expected-fail-message" to test metadata is out of scope of this RFC.
Usually this tag can be added manually by developers after carefully review the
test results and implementations.

### Alternatives Considered

Various alternatives have been considered, e.g. writing a new test to assert the
behavior a baseline wants to assert, or duplicate the test in question as a
legacy web test so that baseline will continue work. Those all would be a
maintanance headache, and are rejected consequently.

## Risks

"expected-fail-message" will only take effect when it presents in the metadata
file. When not presented, the test will behave exactly the same way as before.
It would be at developers' own discretion to use this feature or not. There is a
chance this could be abused, but as the metadata file sits in each vendor's
repository, the impact should be limited.

The existence of this feature should not be used as an implication that subtest
output messages are expected to be stable. When a subtest output message is found
to be not stable, "expected-fail-message" should simply not be used, or if a
vendor still want to assert on the output message, they can choose to revise
the test without sacrifice the usefulness of the test. The original test author
bears no responsibility of this.

This feature will not have any impact to the results presented at wpt.fyi, as
this only changes if a FAIL is expected or not, but not the actual result
itself.
