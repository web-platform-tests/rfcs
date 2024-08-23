# RFC 208: additional crypto and CBOR libraries

## Summary

Several new web APIs require additional cryptography and CBOR libraries to
properly test, for example the Protected Audience Additional Bids feature [0]
uses Ed25519 signatures, the Protected Audience Key-Value services [1] and
Bidding and Auction services [2] use CBOR data encoding and HPKE encryption.
These additional libraries are to support cryptography and the CBOR
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

The Ed25519 library is intended to be used by test code running on wptserve that may receive an Ed25519 private key and message to sign that message, or a public key and signature to verify that signature.

An HPKE JavaScript implementation:
https://github.com/dajiaji/hpke-js

A CBOR JavaScript implementation:
https://github.com/paroga/cbor-js/blob/master/cbor.js

The HPKE and CBOR libraries are used by test code to decrypt and decode data coming from JavaScript APIs to verify their contents, and used by test code to encode and encrypt response data.

## Risks

Users of these libraries may need to update them from time to time if new
functionality or fixes are required. This is likely not a big risk.

The HPKE library proposed may require inclusion of some other dependent
libraries and uses deno to build into a single JavaScript file. Perhaps
it's simplest to commit the single transpiled JavaScript file.

[0] https://github.com/WICG/turtledove/blob/main/FLEDGE.md#623-additional-bid-keys 
[1] https://github.com/WICG/turtledove/blob/main/FLEDGE_Key_Value_Server_API.md#query-api-version-2
[2] https://github.com/WICG/turtledove/blob/main/FLEDGE_browser_bidding_and_auction_API.md
