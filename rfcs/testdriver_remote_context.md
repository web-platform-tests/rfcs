# RFC 91: `testdriver` RemoteContext object

## Summary

Provide `test_driver.RemoteContext` which is a convenient wrapper over
directing commands to another browsing context.

## Details

Lots of testdriver functions take `context` as a final parameter. This
is used when the test file wants to cause some action to happen in a
different browsing context
e.g. `test_driver.delete_all_cookies(context)`. If a context is being
used repeatedly it's tedious and error-prone to have to keep supplying
the context id explictly. Instead it's more convenient to work with an
object with the context id already correctly bound.

`RemoteContext` is an object that wraps the part of the `test_driver` API
which takes a `context` parameter, and calls the underlying API with
a context set at object creation:

```
let ctx = new test_driver.RemoteContext(frames[0]);
ctx.delete_all_cookies()
```

## Risks

As proposed creating the context might be quite verbose. Maybe
`RemoteContext` should be on window instead (or have a shorter name).

This is more code to maintain without adding much additional
functionality.

## References

[PR 29803](https://github.com/web-platform-tests/wpt/pull/29803)
contains a prototype implementation of this.
