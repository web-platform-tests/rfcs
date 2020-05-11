# RFC 53: Introduce more robust mechanism for testharness.js timeout detection

## Summary

Authors of long-running testharness.js tests are able to prevent their tests
from being classified as "timed out," but the available methods of achieving
this are suboptimal. A new mechanism for describing long-running tests would
address much of the drawbacks and give authors more flexibility than they have
today. Specifically: a new value for the `timeout` test metadata describing the
number of subtests which are expected to run.

```diff
-<meta name="timeout" content="long">
+<meta name="timeout" content="400 subtests">
```

## Details

Currently, testharness.js cancels tests and reports their result as "timed out"
if they do not finish running within one of [two possible durations: "normal"
and
"long"](https://github.com/web-platform-tests/wpt/blob/31da90bff3d6fd5cd8169b924125b3253b438ec4/resources/testharness.js#L22-L25).
This behavior has two negative consequences.

Actual testing durations are not as consistent as these two options suggest,
and careful authors are likely to prefer the "long" option to avoid false
negatives. At scale, this tendency may have a non-trivial impact on overall
test execution times (where non-conforming browsers require more time to fail).

Some authors want to run many subtests organized under one logical test.
Although the individual tests may be relatively quick, their number may push
the test's execution time over the limit. Here, authors have to split their
subtests across many files, creating subtests based on anecdotal evidence of
what seems to be "fast enough" to satisfy the duration restriction in their
environment. In the interest of limiting duplication, [this is sometimes
implemented using methods which detract from
readability](https://github.com/web-platform-tests/wpt/pull/11028).

The harness's current restriction serves to detect and recover from cases where
testing has unexpectedly halted. However, the harness currently ignores a
signal that testing is indeed active: the reporting of subtest results. If the
harness restarted its internal "timeout" countdown following every subtest
result, it would be no less capable of detecting timeouts, and authors would no
longer need to manage this expectation. It would also reduce waste (since a
contextual timeout value of, e.g. "10 seconds after the last subtest" is likely
shorter than the static timeout value of "60 seconds").

Many subtests in WPT are generated procedurally-generated. Bugs in test code
(or in browser code) could cause more subtests to be produced than intended.
This could cause the harness to wait far longer than intended (even
indefinitely). To avoid this, the new behavior should be opt-in and doing so
should involve specifying the expected number of subtests. The benefit of a
"timeout" value like "400 subtests" over the existing alternatives ("normal"
and "long") is that the former doesn't require any guesswork about subtest
duration--authors would be configuring in terms that are objectively defined by
the code they've written.

## Risks

As an additional feature of the testing framework, this change would increase
to the amount of time and knowledge required to effectively contribute to WPT.
