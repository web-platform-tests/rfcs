# RFC 208: additional crypto and CBOR libraries

## Summary

Several new web APIs require additional cryptography and CBOR libraries to
properly test. These libraries are to support cryptography and the CBOR
protocols not otherwise supported in JavaScript or Python, namely Ed25519,
HPKE, and CBOR. There are open source libraries commonly available that
implement these protocols and have compatible licenses. This RFC proposes
adding such libraries to the tools/ directory so that web-platform-tests may
exercise and verify proper compatible implementations of these new web APIs.

## Details

We're proposing adding these libraries (or some very similar ones) to the
tools/ directory:

An Ed25519 Python implementation:
https://github.com/pyca/ed25519/blob/main/ed25519.py

An HPKE JavaScript implementation:
https://github.com/dajiaji/hpke-js

A CBOR JavaScript implementation:
https://github.com/paroga/cbor-js/blob/master/cbor.js

## Risks

Users of these libraries may need to update them from time to time if new
functionality or fixes are required. This is likely not a big risk.

The HPKE library proposed may require inclusion some other dependent
libraries and uses deno to build into a single JavaScript file. Perhaps
we can build it in ways similar to other web-platform-tests.
