# RFC 49: Python 3 migration for wptserve


## Summary

This is a proposal to always use the native string type `str` in wptserve APIs in both Python 2 and 3 for a smoother and more ergonomic migration towards Python 2 & 3 compatibility.


## Background

“wptserve has been designed with the specific goal of making a server that is suitable for writing tests for the web platform. This means that it cannot use common abstractions over HTTP such as WSGI, since these assume that the goal is to generate a well-formed HTTP response. Testcases, however, often require precise control of the exact bytes sent over the wire and their timing” -- [web-platform-tests.org](https://web-platform-tests.org/tools/wptserve/docs/introduction.html)

See the high-level [overview](https://docs.google.com/document/d/1_wO8OJ7E1geTiEDyO2ttsuSRCmHDm3bazwDmT9jw04U/edit) of the Python 3 migration effort for the whole WPT project.

**Size of codebase (as of 2020-03-30 revision 4e3194253a):**

*   /tools/wptserve: ~7000 lines
*   /tools/serve: ~1000 lines
*   Custom handlers (test cases): 400+ custom handlers, ~20,000 lines \
This is a rough count of all Python files excluding some directories, which still includes some util scripts: `find . -name "*.py" \! -path "./tools/*" \! -path "./webdriver/*" \! -path "./docs/*" \! -path "./websockets/*" \! -path "./resources/*"`


## Goals

Roughly in the order of importance:

1.  Everything, including wptserve and custom handlers, must continue to work in both Python 2 and 3 consistently. (This was discussed and agreed upon at TPAC 2019.)
1.  Custom handlers must continue to be able to read and write raw bytes from requests and to responses, including headers and body. wptserve must not change the byte sequences in any way; no errors should be raised if the byte sequences do not have a valid encoding.
1.  _Optional_: minimize code change to custom handlers, and consequently minimize the burden on test authors in the future by imposing fewer new requirements. (Whether to strive for this goal will largely affect which option we choose.)


## Root problem (or; why is this hard?)

In both Python 2 and 3, unprefixed string literals have the `str` type. However, the meaning of the `str` type has changed: in Python 2 it is a raw byte sequence, whereas in Python 3 it is a Unicode text string. The rationale behind the change was to move to a Unicode-by-default world: arguably most people who used `str` in Python 2 were probably dealing with text and should use Unicode, so this decision minimized the impact on code handling human-readable text. However in WPT we actually intended to use byte sequences (see [background](#background)).

This pain is further compounded by the choice of types in some parts of the Python 3 standard library. The low-level [`http`](https://docs.python.org/3/library/http.html) library takes and returns the `str` type (i.e. Unicode string) almost everywhere except the body (see the [type](https://github.com/python/typeshed/blob/master/stdlib/3/http/client.pyi) [annotations](https://github.com/python/typeshed/blob/master/stdlib/3/http/server.pyi)). This is convenient for existing code from Python 2 but semantically different and makes it difficult to send raw bytes on the wire. On the upside, the standard library carefully and intentionally uses an 8-bit isomorphic encoding (latin-1/ISO-8859-1) when appropriate (e.g. [headers](https://github.com/python/cpython/blob/8c3ab189ae552401581ecf0b260a96d80dcdae28/Lib/http/client.py#L220)) to allow arbitrary byte sequences:

*   Getting bytes out: “encode” the return value with latin-1
*   Passing bytes in: “decode” the byte sequence with latin-1


## Options


### [Recommended] Keep the ergonomics: always use str

I would argue that the most ergonomic approach (for test authors) is to **always use unprefixed string literals and the `str` type** in the majority of cases regardless of Python 2 or 3 despite the different semantics for the following reasons:

*   It avoids having to modify hundreds of custom handlers (most likely manually).
*   It does not impose additional burden on test authors (e.g. remembering to prefix string literals with b even if it is a no-op in Python 2, which will still remain the default for a while). In addition, any additional requirement on types would be impossible to fully enforce in theory since Python is dynamically typed.
*   This is what the Python community has chosen to do, so it aligns well with the standard library without additional en/decoding that depends on the version of Python. Custom handlers do not need `ensure_str`/`maybe_encode` or have different code paths for Python 2 and 3.


#### Detailed changes

To achieve this, we will need some encoding magic to store arbitrary byte sequences in `str` in Python 3. This will be similar to what the standard library does, including but not limited to:

*   In the request handler, we need to explicitly use latin-1 encoding for [`cgi.FieldStorage`](https://github.com/web-platform-tests/wpt/blob/5eb4894a68051e04b14fe471da38dd8817e417ed/tools/wptserve/wptserve/request.py#L309) instead of the default utf-8 encoding to allow non-UTF-8 values in Python 3. (In Python 2, this class does not have encoding.)
*   Stop forcing the headers to be byte sequences in both [Request](https://github.com/web-platform-tests/wpt/blob/5eb4894a68051e04b14fe471da38dd8817e417ed/tools/wptserve/wptserve/request.py#L378) and [Response](https://github.com/web-platform-tests/wpt/blob/1e0302fe4048bef22ad72168bd754fee8d7f2ca5/tools/wptserve/wptserve/response.py#L320). Instead, directly pass the `str` headers from and to the standard library. The [`parse_headers()`](https://github.com/python/cpython/blob/8c3ab189ae552401581ecf0b260a96d80dcdae28/Lib/http/client.py#L200) function in Python 3 uses latin-1 so it can preserve any byte sequence.
*   [Authentication](https://github.com/web-platform-tests/wpt/blob/5eb4894a68051e04b14fe471da38dd8817e417ed/tools/wptserve/wptserve/request.py#L611) needs to decode `username` and `password` with latin-1 to make sure they are always `str`.
*   Optionally, we might want to add additional getters and setters for headers, which are guaranteed to return and take byte sequences to provide additional clarity and control if the test author desires. Such getters and setters will use `encode("latin-1")` and `decode("latin-1")` respectively under the hood. Alternatively, we could add a type switch to the existing setters to automatically decode if `bytes` are passed in, but custom getters would always be required to receive bytes out.


#### Outcomes and risks

With this approach, most custom handlers should continue to work in both Python 2 and 3. For example, all of the following strings can be compared against `request.headers` or `request.auth`, and accepted by `response.headers` in both Python 2 and 3:

*   `"ASCII strings"`
*   `"\x32\042"`
*   `b"\x32\042".decode("latin-1")`
*   `"你好\u1234".encode("utf-8").decode("latin-1")` \
Note: the u prefix is needed for Unicode literals to work in Python 2

There are two major caveats:

*   Printing and logging will be largely broken for non-ASCII characters in Python 3 -- strings will need to be round-tripped before printed, e.g. `message.encode("latin-1").decode("utf-8")`).
*   Custom handlers that currently explicitly use the b prefix for string literals will break in Python 3 -- they need to either remove the prefix or use the new getters and setters for `bytes`. There are only a handful of instances, so this seems tractable.

Note that this might be confusing and counter-intuitive for those with more understanding of Python encoding as the API of wptserve will have two different semantics (bytes vs strings) depending on the version of Python. However the situation will be similar to the standard library and this quirk of coercing byte sequences to strings is a [common hack](#references) in Python 3.


### [Alternative] Keep the semantics: always use byte sequences

An alternative approach would be to keep a consistent and semantically correct encoding model. That is to **always use byte sequences: `str` in Python 2 and `bytes` in Python 3**.


#### Detailed changes

*   Keep the `ensure_{str,bytes}/maybe_{encode,decode}` functions, and double check that they are used everywhere in wptserve.
*   Prefix string literals in custom handlers with `b` when necessary, which needs to be done manually (some strings should not be prefixed). Note that we could add a hack to setters to accept both `str` and `bytes`, but on the getter side there is little we could do -- we don’t know whether the returned value will be used together with `bytes` or `str`, and things like `dict.get()` will be particularly tricky.


#### Outcomes and risks

*   Printing and logging will also need to be changed in Python 3. Custom handlers need to have a special code path for Python 3 to encode the strings.
*   We need a new coding guideline to make sure string literals are prefixed with `b` correctly to stay Python 2 and 3 compatible. We can't simply require all strings to be prefixed with either `b` or `u`, as sometimes native strings are desired (notably some standard library APIs that take native strings in both Python 2 and 3).

To make it clear, I do not think the purity of the encoding model justifies the scale of the change and ongoing maintenance.


## Side notes

**`Response`** already [handles encoding of the content explicitly](https://github.com/web-platform-tests/wpt/blob/edc6681c45e1a25e36e99dcd90dada97ce086dae/tools/wptserve/wptserve/response.py#L27).

**Stash** is the key-value storage for custom handlers. Keys are used to [create uuid.UUID](https://github.com/web-platform-tests/wpt/blob/5eb4894a68051e04b14fe471da38dd8817e417ed/tools/wptserve/wptserve/stash.py#L150), which are expected to be `str` in both Python 2 and Python 3. Most, if not all, custom handlers use `str` keys. Values are stored directly in a Python dictionary and can be any Pickleable-type.


## References

For reference, here is how some popular Python libraries deal with HTTP headers:

*   [Werkzeug](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.Headers) (the library used by Flask) always takes and returns `str` by default, unless ``as_bytes`` is True. If `bytes` is given to the getter, it will be encoded using latin-1 (see the changelog for v0.9). This is effectively Option A with the optional setters and getters.
*   [Requests](https://requests.readthedocs.io/en/master/api/#behavioural-changes) enforces “native stringsâ (`str`) almost everywhere. It uses UTF-8 instead of latin-1 to automatically encode keys (header names), but the encoding doesn’t quite matter here because HTTP header names are supposed to be ASCII.
