# RFC #78: Revamp idlharness.js

## Summary

At least:
- Determine dependencies as a build step, since this is tedious manual work and it's not possible to know if any dependency is missing in the case of mixins.

At most:
- Delete `idlharness.js` and generate all tests, either as a build step or as a wptserve feature.

## Details

idlharness.js is now only used via `idl_test()`, so the internals can be refactored.

## Risk

Many tests and subtests renamed, it will be hard to keep track of any bugs filed against the existing tests.
