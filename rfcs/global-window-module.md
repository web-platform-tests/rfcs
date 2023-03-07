# RFC 133: Add support for `window-module` as JavaScript global

## Summary

Add a new global called `window-module` so that other resources can be imported
as modules on tests running on `window`.

##  Details

`tools/manifest/sourcefile.py`'s `_any_variants` will get `"window-module"`,
and `tools/serve/serve.py` will get `WindowModuleHandler` that loads the script
as a module.

## Risks

Since module scripts can have top level await, some people may try awaiting any
promise to setup the precondition before starting tests, instead of using
`promise_setup()`. But such risk already exists since one can write a full HTML
to achieve the same and there are other module globals e.g.
`dedicatedworker-module`.
