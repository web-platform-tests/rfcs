# RFC ..: Remove default and negated global support in *.any.js

## Summary

https://web-platform-tests.org/writing-tests/testharness.html#multi-global-tests has a confusing syntax for specifying globals. E.g., `global=window` is equivalent to `global=window,dedicatedworker` and `global=jsshell` is equivalent to `global=window,dedicatedworker,jsshell`.

## Details

1. Update all `*.any.js` tests that use `// META: global=` to list all the relevant globals explicitly when possible. (Some necessarily rely on `!default` or `!` (negation).
2. Atomically:
   1. Remove support for `default` and `!`.
   1. Update all remaining `*.any.js` tests that use `!` to list all the relevant globals explicitly.
   
https://github.com/web-platform-tests/wpt/issues/23111 tracks this work.
