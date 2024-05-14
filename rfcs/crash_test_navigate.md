# RFC 193: Navigate after Crash Tests

## Summary

Navigate to about:blank after successfully loading the test URL in
crash tests.

## Details

Add a step to the end of crash tests where the browsing context that
loaded the test is navigated to about:blank.

This is because we have seen cases where the crash the test is trying
to detect doesn't happen until a subsequent navigation. That kind of
crash is hard to handle in web-platform-tests: if the test happens to
be the last one in a group it may be missed entirely, and because we
want to make it possible to import tests without blocking on fixing
the bugs they reveal, it's important to be able to annotate a crash as
related to a specific test so the expectation metadata can be set
correctly; if the actual crash happens at the start of the following
test this doesn't work because we've already recorded an incorrect
PASS status for the previous test. It's also helpful for humans to see
the test that's actually failing when debugging the problem.

## Risks

This will decrease the performance of crash tests. However since
they're few in number and generally fast compared to other test types,
this seems like an acceptable tradeoff.

Since about:blank is a special URL it's possible that navigating to
that is triggering different codepaths to navigating to an actual
server-provided resource. We could fix this by navigating to a blank
page on the server, but this would likely have even worse performance
implications. We should consider this option if we find there are
cases not caught by this approach that would be caught when navigating
to a http URL.

This won't cover all possible cases of delayed crashes; in general
that's an impossible problem to solve. However it is an incremental
improvement over the status quo.
