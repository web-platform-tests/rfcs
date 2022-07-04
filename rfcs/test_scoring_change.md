# RFC: Change results aggregation on wpt.fyi and visualization on Interop-20** views

## Summary

The aggregate scoring that is displayed in the results view of wpt.fyi is
inaccurate in some aspects. This proposal will describe a change to the way
that test results are aggregated, as well as a change to how those results are
displayed for Interop 2022 scoring purposes.

## Details
There are a number of aspects to this change. Each subsection will describe
these aspects in detail.
### 1. Remove harness status from subtest aggregation

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

### 2. Display additional information if a harness error exists.

Although it seems intuitive to simply disregard the harness status when
calculating tests, there are instances where a harness error will occur, which
causes a number of subtests to not be executed. In some circumstances, it is
possible that a harness error will occur, allowing for some subtests to run
and pass as expected, but some subtests to be not run and not be registered,
as they are not present during aggregation. This has the possibility of hiding
the scope of failure for a specific test in this scenario. The solution
proposed is to mark these tests with harness errors with some additional
visual notice that this a harness error occurred. This is to signify that
the results should not be taken at face value and engineers should investigate
the test further.

### 3. Implement optional display of Interop-20xx scores for labelled tests

Currently, wpt.fyi displays a fraction of the number of subtests that pass and
the number of subtests that exist for each test or every test that exists in a
directory. This view is not necessarily indicative of Interop scores and it
can be unclear to engineers what aspects are affecting scores easily. The
proposed change is to have these Interop views better match their actual score
weights for easier understanding. This can be handled by aggregating by tests
rather than by subtests.

Passing percentage for a single test = `passing subtests / total subtests`

Passing percentage for directories containing multiple tests =
`sum of passing percentages of tests in directory / total number of tests in directory`

This will be implemented by adding a new query parameter, `view`. This can have
two valid values: `subtest` and `interop`. `subtest` will represent the current
view that exists on wpt.fyi and will be enabled by default. `interop` will
display a view that more closely mirrors the Interop-20** scores.

**Note**: In the interest of avoiding scored comparisons being visible outside
the tests that have been agreed upon for Interop-20**, this `interop` view will
only be available when viewing tests with an `Interop-20**` test label. The
`interop` view will not function outside of results without this label.

## Risks
* This change will cause a discrepancy in the number of subtests when comparing
runs with old summary files and new summary files together, as the harness
status passing has been artificially inflating some subtest numbers. This is a
rework of the current process for aggregating and summarizing test results,
so older test runs will still be aggregated with the harness status.
