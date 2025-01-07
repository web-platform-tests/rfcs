# RFC XXX: Migrate Python vendoring to vendoring

## Summary

We should use [vendoring](https://pypi.org/project/vendoring/) to re-do all of our Python vendoring,
directly into `third_party` (crudely: `pip install -t tools/third_party ...`),
and then move to a generated `requirements_vendor.txt`.

[Implementation](https://github.com/web-platform-tests/wpt/pull/49752).


## Background

We currently have a directory in WPT, called `tools/third_party`,
which contains various bits of vendored code,
which we copy into our tree primarily to
[avoid Mozilla having to update their partial PyPI mirror](https://github.com/web-platform-tests/rfcs/issues/82#issuecomment-2462913061).

This currently contains copies of a number of Python libraries as well as pdf.js,
currently mostly imported via `git-subtree`.

It's difficult to know what exact versions we have vendored,
and it is hard to update them,
and especially hard to know when we no longer need libraries we vendor
(especially those we vendor only as dependencies of other libraries).

The Python code is added to `sys.path` via
[`tools/localpaths.py`](https://github.com/web-platform-tests/wpt/blob/59ee38f559b1bb85b9474b4e870bf70183f50414/tools/localpaths.py).


## Details

Similar to the [implementation](https://github.com/web-platform-tests/wpt/pull/49752/commits),
this is split into four sections:

### Move `pdf.js`

`vendoring` as a tool expects the vendored directory to be fully under its control;
as a result, we need to move the singular non-Python dependency out of the directory.

This RFC proposes that we move `pdf.js` from `tools/third_party/pdf_js` to `tools/third_party_js/pdf_js`.


### Re-vendor everything we currently have vendored

To make it easy to bisect any regressions from this change,
we should start off by initially just re-vendoring everything,
just now using the `vendoring` tool,
listing everything in `tools/requirements_vendor.txt`.

This causes one major change:
it removes everything except the contents of the distribution packages,
thus removing our vendored copies of the documentation,
test suites,
CI scripts,
and everything else they happen to have in their repos.

This results in everything being importable from the `tools/third_party` directory,
thus `localpaths` can be simplified to only add that directory to `sys.path`.

This causes us to have to make two other changes to our Python:

 1. Change the path to the `tooltool` module in `tools/wpt/android.py`.

 2. Explicitly list `pytest_asyncio` as a plugin in `webdriver/tests/conftest.py`.

The latter of these is necessary because we no longer discover the entry-point of the vendored copy.

The only other notable change is we now maintain a `tools/third_party_patches`,
which contains all the patches we are applying to what we're importing.
This is currently only `pywebsocket3`, with changes in
[efa4a99b8d](https://github.com/web-platform-tests/wpt/commit/efa4a99b8dde1d9ab572efb9e1757e6900289bed)
([upstream issue #38](https://github.com/GoogleChromeLabs/pywebsocket3/issues/38)) and
[3bd9ff9f8a](https://github.com/web-platform-tests/wpt/commit/3bd9ff9f8a175b554cc2e78f80b9d28fff73f66f)
(no upstream issue).

Otherwise, this change is roughly a two million line removal from WPT,
because we import a lot of documentation and tests.


### Downgrade html5lib to 1.1

Currently we have html5lib imported from [f4646e6](https://github.com/html5lib/html5lib-python/commit/f4646e6ed4eeb9780f67d2083d0c09c8fffbec53),
which is the commit after 1.1's release,
changing the version number to 1.2-dev.

We should downgrade html5lib to 1.1,
thus allowing us to vendor the version released on PyPI.


### Use `uv pip compile` to build the dependency tree

Finally, we can move to just listing everything we actually depend on in a `requirements_vendor.in`
and generate our `requirements_vendor.txt` from that.

This is where risks start to come in:
this removes a number of vendored libraries,
because we no longer need them.


## Risks

### External code relying on paths within `tools/third_party`

One could, hypothetically, have someone doing:

```python
sys.path[0:0] = ["path/to/wpt/tools/third_party/pytest"]
```

and have it expected that this works to make `pytest` importable.

Making these changes would break this,
as `tools/third_party` is now the only path added to the module search path.


### External code relying on dependencies we no longer require

By removing projects like `attrs`,
which we no longer use,
it could turn out that other external code was relying on the WPT vendored copies being on the `sys.path`,
rather than listing them as dependencies itself.


### Lowering the barrier to vendoring new projects may make us overly reliant on vendoring

As a result of making it easier to vendor projects,
we could end up with people adding much more vendored code.

This is problematic both insofar as we become distributors of the contents of the distribution package,
with the potential legal implications of that,
and makes us potentially more inclined to distribute an every larger number of packages.

It also potentially decreases the motivation for Mozilla to make it simpler to update their PyPI mirror,
thereby effectively perpetuating our vendoring further into the future.


## Further work

The most obvious follow-up item to this is "update all of our vendored projects",
and while we normally don't file RFCs for dependency updates,
it is potentially higher risk when we haven't done this in years.
