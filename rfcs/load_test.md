# RFC 33: Crash Tests

## Summary

Add a test type in which the test document is simply loaded. The test
passes if this occurs without encountering a fatal error e.g. a crash.

## Details

Load Tests are static tests typically used for cases where browsers
are known to have a bug such as a crash or a leak. They differ from
other tests such as reftests and testharness tests in that there is no
pass condition other than the page loading successfully. In vendor
infrastructure these tests may also be run with additional
instrumentation e.g. address sanitizing, making the pass condition
more stringent. The lack of an explicit pass condition makes it easier
to write these tests than tests where a detailed understanding of the
correct behaviour is required, making them suitable for autogeneration
e.g. as the output of a fuzzing tool.

The infrastructure changes are as follows:

* A new test type `crash` recognised by the manifest and assigned to
  any test file with a name containing `-crashtest` immediately before
  the extension or in a subdirectory called `crashtests`.

* A new executor type with the following behaviour:
  - Load the test URL
  - If there is no `class=wait` on the root element, mark the test as
    passed when the load is complete, all fonts have finished loading,
    and the rendering is stable.
 - If `class=wait` appears on the root element, mark the test as
   complete once the previous conditions are met and that class has been
   removed.

## Risks

Adding this test type may encourage test authors to write crashtests
for cases where a more useful cross-browser test would have a real
pass condition.

Crashes and other issues caught by these tests may not be sufficiently
common between browsers to make sharing these tests a good use of
resources. However there's data showing cases where a tests that crash
one browser also crash others. Therefore this concern seems less
serious than it may at first appear.
