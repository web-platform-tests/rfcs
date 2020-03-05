# RFC 42: Add an optional QUIC server to wpt

## Summary

Add an *optional*, *off-by-default* [QUIC](https://quicwg.org/) server to wpt,
implemented with [aioquic](https://github.com/aiortc/aioquic) in Python 3. This
server is used to test web APIs that rely on the QUIC protocol (e.g.
[WebTransport](https://wicg.github.io/web-transport/)). If the server is not
enabled, `wptrunner` will skip the tests that rely on QUIC.

QUIC custom handlers use a new filename flag `.quic` (`.quic.py`). And a meta
tag is included by tests that require the QUIC server for easy filtering.

This RFC focuses on the high-level architecture and dependency management to
enable prototyping the QUIC server. The server APIs are not yet finalized, but
we commit to filing another RFC to discuss and finalize the API design once it
is ready.

See the [original issue][original-issue] for more background.

## Details

### Requirements

1.  The QUIC server requires decent cryptography support; pure Python isn't
    sufficient.
2.  The server needs to be scriptable, and the custom handlers should live in
    the wpt repo next to test files, so that they can be updated together.
3.  An async programming model is preferred.

### Architecture

The QUIC server has a standalone CLI, and is managed by `wpt serve` as a
subprocess. Configurations are stored in `config.json` (same as `wptserve`), and
`wpt serve` is responsible for passing the relevant configurations (e.g. port
numbers) to the QUIC server via command line flags. `wpt serve` handles the
error gracefully if the QUIC server cannot be started.

The QUIC server is written in Python 3 and makes no attempt to support Python 2.
Its entrypoint will have a shebang that explicitly requires `python3`. An
implication is that the QUIC server will be relatively self contained, without
dependencies on other modules in wpt that are currently Python 2-only.

Code for the QUIC server itself will live in `tools/quic`.

### User interface

For **test writers**: the QUIC server will support Python custom handlers
similar to `wptserve`. The APIs are not yet finalized and will likely be
different. To more easily distinguish the two, we add a filename flag to QUIC
custom handlers (`.quic.py`). These handlers will be ignored by `wptserve`.
Tests that require the QUIC server need to be marked with a meta tag
(tentatively `<meta name="quic" content="true">` and `// META: quic=true`). This
information will be extracted by `wpt manifest` into `MANIFEST.json`, so that
`wptrunner` can skip the tests that require the QUIC server easily. A lint rule
will be added to require this meta tag if `.quic.py` appears in the file, even
though it cannot prevent all accidental dependency of the QUIC server (e.g.
through a resource script).

For **test runners**: the goal is to not introduce any observable changes to
users not yet concerned with QUIC or WebTransport. All existing tests will
continue to run without any infra changes, and all new QUIC-dependent tests will
be skipped by default. To enable the QUIC server, users need to pass a flag to
`wpt run` (or `wpt serve`). `wpt` front-end will then attempt to download the
the dependencies (wheels) from PyPI into `_venv`; maintainers of infrastructures
where PyPI cannot be used need to manage dependencies themselves (see the next
section).

### Notable dependencies

* [sys] Python 3 (>=3.6)
* [py] aioquic
    * [py] cryptography
        * [sys] openssl (>=1.0.1, already required by [HTTP/2](http2.md))
    * [py] pylsqpack

All of the Python dependencies above have native extensions, which requires
users to use a platform supported by the prebuilt wheels from PyPI or build and
distribute the wheels themselves (e.g. in Chromium through
[cipd](https://chromium.googlesource.com/chromium/src/+/master/docs/cipd.md)).

The source code of all dependencies will also be vendored into
`tools/third_party` to allow building from source if needed, even though we
there is no plan for that yet.

All Python dependencies are BSD-licensed.

### Alternatives considered

We have also [considered][original-issue] writing the QUIC server in other
languages (e.g. Go) and publishing a statically linked binary that would be
downloaded by `wpt` similar to how WebDriver binaries are installed. This would
incur a smaller overhead on the wpt repo in terms of new dependencies, but would
make writing [scriptable custom handlers](#requirements) harder -- we would need
to build an interface between the server and the handler scripts (which will
likely be in a different language, like Python).

Another alternative that avoids mixing Python 2 and 3 is to migrate the whole
`wptserve` to Python 3, but that work has a much bigger scope.

## Risks

The change required in `wpt` is limited, and the QUIC server itself is
standalone, so the maintenance burden of the code base will be low. The
Ecosystem Infra team at Google will be responsible for maintaining this
integration point.

A new meta tag means a new field in the manifest JSON (only needed for tests
that use the QUIC server), which should be backward compatible.

The biggest concern moving forward is the management of the binary dependencies.
Whenever a dependency is updated, we will need to notify all vendors who opt
into the QUIC server so that they can prepare the new wheels in their infras.

[original-issue]: https://github.com/web-platform-tests/wpt/issues/19114
