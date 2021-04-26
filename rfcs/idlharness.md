# RFC #78: Revamp idlharness.js

## Summary

At least:
- Determine dependencies as a build step, since this is tedious manual work and it's not possible to know if any dependency is missing in the case of mixins.

At most:
- Delete `idlharness.js` and generate all tests, either as a build step or as a wptserve feature.

## Details

idlharness.js is now only used via `idl_test()`, so the internals can be refactored.

Kinds of dependencies currently:
- Partial definition depends on the original definition.
- Dictionary inheritance chain is needed to determine if the dictionary is a [JSON type](https://heycam.github.io/webidl/#dfn-json-types), which in turn is used to test `toJSON` operations of *interfaces*. But this doesn't really make sense since attributes can't be dictionaries??
- Interface inheritance is used to test if the prototype chain is correct and when testing objects
- `internal_add_dependency_idls` adds dependecies on the types used for attributes, probably used for namespace attributes and when testing objects.

## Risk

Many tests and subtests renamed, it will be hard to keep track of any bugs filed against the existing tests.
