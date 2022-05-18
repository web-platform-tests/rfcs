# RFC: Change results aggregation and visualization on wpt.fyi to increase accuracy and utility

## Summary

The aggregate scoring that is displayed in the results view of wpt.fyi has
consistently been inaccurate in some aspects. This proposal will describe a
change to the way that test results are aggregated, as well as a change to
how those results are displayed.

## Details
There are a number of aspects to this change. Each subsection will describe
these aspects in detail.
### Remove harness status from subtest aggregation

When interpreting the results of test suite runs, a status is present that
represents either the overall test's status or the "harness status", which
represents whether any harness errors were present for `testharness.js` test
runs. The current results aggregation method counts this harness status as a
separate subtest itself, which can inflate the overall passing percentage of
a test. There are many instances of a test outright failing its only subtest,
but the test is weighted as 50% passing because there were no harness errors
[(example)](https://wpt.fyi/results/audio-output/selectAudioOutput-sans-user-activation.https.html?run_id=5642791446118400&run_id=5668568732532736&run_id=5739999172493312&run_id=5735075831349248).
The solution proposed is to no longer aggregate this harness status into the
subtest totals.

### Count test as a failure if a harness error exists

Although it seems intuitive to simply disregard the harness status when
calculating tests, there are instances where a harness error will occur, which
causes a number of subtests to not be executed. In some circumstances, it is
possible that a harness error will occur, allowing for some subtests to run
and pass as expected, but some subtests to be not run and not be registered,
as they are not present during aggregation. This has the possibility of hiding
the scope of failure for a specific test in this scenario. The solution
proposed, although not perfect, is to mark these tests with harness errors
as 0% passing, regardless of subtest results.

_Note: Individual subtest results will still be visible normally even with_
_this change. This only affects the total percentage that the test counts as_
_passing._

### Display percentages to represent the fraction of tests passing.

Currently, wpt.fyi displays a fraction of the number of subtests that pass and
the number of subtests that exist for each test or every test that exists in a
directory. These fractions can get large and be hard to quickly parse their
meaning, and a single test with many subtests is heavily weighted. The
proposed change is twofold: display this number as a percentage, and instead
count each test as 1 total unit, with its pass rate counted as the percentage
of subtests that pass. Directories will aggregate based on this each test pass
percentage rather than the accumulation of every subtest.

Passing percentage for a single test = `passing subtests / total subtests`

Passing percentage for directories containing multiple tests = `sum of passing percentages of tests in directory / total number of tests in directory`

For example, if a test has 5 out of 10 subtests that pass, the it will now be
displayed in its representing cell as `50%` rather than `5 / 10` for that run.
Additionally, if we have a directory containing 3 tests and 2 of those tests
pass 100% of their subtests and the last test passes 50%, the directory will
display `83.3%`.

### Display the number of tests in a directory next to the directory name

The current implementation of displaying fractions of subtests has a benefit
of signaling where a bulk of test information is located and how much to
expect inside of that directory. Displaying percentages causes some loss of
this information. To mitigate this, the proposal is to instead display test
totals next to a directory name. This will be all tests that exist in the
directory as well as the number of tests that exist in all subdirectories
within that directory.

### Display test results in single test cells rather than percentages

If a test has no subtests and only a pass/fail result, that result status
( `PASS` , `FAIL` , `TIMEOUT` , etc.) will be displayed in the cell that
represents that test rather than the existing `0 / 1` or `1 / 1` . The user
will no longer have to drill down into the test to view the status message in
these scenarios.

## Risks
* This change will cause a net loss in the percentage of passing
tests/subtests, as the harness status passing has been artificially inflating
some subtest numbers.
* Some tests might pass most or all subtest but encounter a harness error,
which will cause the entire test suite to be counted as a failure. This will
also result in a lower rate of overall passing percentages.
* This change will require a rework of the current process for aggregating and
summarizing test results, so older test runs will still be aggregated with the
harness status and display inflated passing percentages.
* Tests will be weighted equally, regardless of the number of subtests that
exist in that test.
