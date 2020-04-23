# RFC 52: Remove default and negated global support in \*.any.js

## Summary

https://web-platform-tests.org/writing-tests/testharness.html#multi-global-tests has a confusing syntax for specifying globals. The default set of globals is always included implicitly, e.g., `global=window` is equivalent to `global=window,dedicatedworker` and `global=jsshell` is equivalent to `global=window,dedicatedworker,jsshell`.

This RFC will make these changes:

1. Remove `default` (though keep `window,dedicatedworker` as the default if `// META: global=` is not present).
2. Remove `!` (negation). Instead test developers are expected to either rely on the default or explicitly list the globals they want.

A test supposed to run in dedicated and shared workers would use `dedicatedworker,sharedworker` and not `!window,sharedworker`, `!window,worker,!serviceworker`, etc.

## Details

1. Update all `*.any.js` tests that use `// META: global=` to list all the relevant globals explicitly when possible. (Some necessarily rely on `!default` or `!`.)
2. Atomically:
   1. Remove support for `default` and `!`.
   1. Update all remaining `*.any.js` tests that use `!` to list all the relevant globals explicitly.
   
https://github.com/web-platform-tests/wpt/issues/23111 tracks this work.

## Risks

Downstream users could have reimplemented some of the infrastructure logic and end up running tests in more globals than they should (by always including the default globals).
