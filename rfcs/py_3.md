# RFC 69: Switch web-platform-tests to Python 3

## Summary

This RFC sets out the overall plan and timetable for the Python
2->Python 3 transition. It is anticipated that further details on the
transition will be set out in other RFCs as appropriate.

## Details

 1. **November / December 2020** XXX add actual date - Python 3 becomes default on GitHub CI

    Details: https://github.com/web-platform-tests/rfcs/pull/65/

    Prerequisites:

     * All existing Python code updated to work with Python 3
     * No regressions in test behaviour or harness functionality in Python 3
     * A --py2 flag is added to `wpt` to opt-in to running in Python 2

    After this time unittests will still be run as Python 2 and Python
    3 but full test runs and PR test runs will be Python 3 only.

    Handler code will be expected to continue working in Python 2.7.


 1. **January 1st 2021** - Vendors are expected to have completed their transition to Python 3

    After the upstream transition to Python 3, it is assumed that
    vendors will wait a few days to ensure that the change sticks and
    then arrange to update their CI systems to match upstream.

    While upstream is mainly running Python 3 but vendors are still
    running Python 2 they may submit changes that only work with
    Python 2, requiring post-hoc fix-up. Therefore in this period
    careful monitoring of Python changes will be needed.

    After this date `wpt` will run with Python 3 by default and `wpt
    --py2` will be needed to get the old behaviour. The `--py3` flag
    will be a no-op.

 1. **February 1st 2021** - Python 2 no longer run on CI

    After January 1st it will be assumed that everyone has converged
    on Python 3 and Python changes will not be monitored for 2.7
    compatibility. However unittests will continue running on 2.7
    until February, in case there is an unexpected need to revert the
    changes. In this period we will not intentionally accept
    changes that regress Python 2 (e.g. removing six usage).

    After February 1st all testing on Python 2 will be disabled and we
    will accept patches to remove Python 2 support.

    The Python version flags will be removed from `wpt`.
