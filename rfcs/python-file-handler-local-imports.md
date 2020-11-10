# RFC 68: Make local imports in Python file handlers root-relative

## Summary

Require local imports in Python file handlers to be specified relative to the
`docroot` of wptserve, e.g.:

```
html/
└── dom/
    ├── handler.py
    └── helper.py
```

Would require `handler.py` to import `helper.py` via:

```python
from html.dom import helper
```

## Background

[Python file handlers] are Python files that `wptserve` will execute in
response to requests matching a corresponding URL. Currently Python file
handlers are able to import local helper scripts from the same directory they
are in, e.g.:

```
html/
└── dom/
    ├── handler.py
    └── helper.py
```

Here, `handler.py` can import `helper.py` via:

```python
import helper
```

This behavior is currently supported by [mutating sys.path and
sys.modules][mutating-code] when loading a file handler. Unfortunately this
mutation [does not actually work][filehandler-bug]:

1. The copying of `sys.modules` is not sufficient, so helper scripts collide by
   name (no matter where in the tree they are).
2. The copying of `sys.modules` can also corrupt underlying state, leading to
   import errors in Python 3.
3. `wptserve` is actually threaded, so we are also messing with `sys.path` and
   `sys.modules` in a threaded manner - this amazingly hasn't caused us pain so
   far in Python 2 but is definitely dangerous.

An effort was made to [fix these problems][filehandler-fix-1] whilst keeping
the same import syntax, but required brittle changes relying on deep Python
internals, so instead this RFC proposes changing the local-import syntax.

## Details

The core changes that this RFC captures are to:

1. Remove the adding of the local directory for a given Python file handler to
   `sys.path`,
2. Remove the attempted mutation of `sys.modules`, and
3. Add the wptserve `docroot` (usually the WPT root directory) to `sys.path`.

This has a number of knock-on effects. Most prominently, test authors will now
have to reference local helper files by their full path:

```python
from html.dom import helper
```

If the helper script is in a directory that contains a hyphen (e.g.
`css/css-animations/helper.py`), test authors will have to use
`importlib.import_module`:

```python
import importlib
helper = importlib.import_module("css.css-animations.helper")
```

Until we reach Python3-only (and thus can rely on [PEP 420]), test authors will
also have to make sure there is a path of `__init__.py` files from the WPT root
to the helper file, e.g.:

```
__init__.py
html/
├── __init__.py
└── dom/
    ├── __init__.py
    ├── handler.py
    └── helper.py
```

### Knock-on impact of root `__init__.py` on unittests

As noted above, until we get to Python3-only this RFC requires a root
`__init__.py` file to be added to WPT. This causes some problems for the WPT
unittests, which previously relied on [pytest behavior] to 'magically' add the
root WPT directory to `sys.path` and allow them to do e.g.

```python
from tools.ci.tc import decision
```

With this RFC, these unittests all need to change to explicitly add the root to
the path, e.g.:

```python
here = os.path.dirname(__file__)
root = os.path.abspath(os.path.join(here, "..", "..", "..", ".."))
sys.path.insert(0, root)

from tools.ci.tc import decision
```

Once we have PEP 420 available, these can all be cleaned up and restored to
their previous behavior.

## Risks

* The new syntax is undeniably harder to use than the previous syntax, and so
  may annoy test authors or push them away from using python file handlers.
   * In particular we have a lot of directories in WPT that use hyphens
     (approximately 60% of unique directory paths in WPT contain a hyphen).
   * Albeit note that this only applies if one is using a local helper file,
     which few file handlers actually do.

[Python file handlers]: https://web-platform-tests.org/writing-tests/python-handlers/index.html
[mutating-code]: https://github.com/web-platform-tests/wpt/blob/09dd5ef1d47633394f6e368ef62fd4f665cf18a6/tools/wptserve/wptserve/handlers.py#L290
[filehandler-bug]: https://github.com/web-platform-tests/wpt/issues/25678
[filehandler-fix-1]: https://github.com/web-platform-tests/wpt/pull/26111
[PEP 420]: https://www.python.org/dev/peps/pep-0420/
[pytest behavior]: https://docs.pytest.org/en/stable/pythonpath.html#prepend-and-append-import-modes-scenarios
