# RFC 208: additional crypto and CBOR libraries

## Summary

Several new web APIs require additional cryptography and CBOR libraries to
properly test, for example the [Protected Audience Additional Bids feature](https://github.com/WICG/turtledove/blob/main/FLEDGE.md#623-additional-bid-keys )
uses Ed25519 signatures, the [Protected Audience Key-Value services](https://github.com/WICG/turtledove/blob/main/FLEDGE_Key_Value_Server_API.md#query-api-version-2) and
[Bidding and Auction services](https://github.com/WICG/turtledove/blob/main/FLEDGE_browser_bidding_and_auction_API.md) use CBOR data encoding and HPKE encryption.
These additional libraries are to support cryptography and the CBOR
protocols not otherwise supported in JavaScript or Python, namely Ed25519,
HPKE, and CBOR. There are open source libraries commonly available that
implement these protocols and have compatible licenses. This RFC proposes
adding such libraries to the tools/ directory (for the Python library) and
to a [<spec>/third_party/ directory (for the JavaScript libraries)](https://github.com/web-platform-tests/rfcs/issues/46#issuecomment-587707539) so
that web-platform-tests may exercise and verify proper compatible
implementations of these new web APIs.

## Details

### We're proposing adding these Python libraries to the tools/ directory:

These are all self-contained (no dependencies) pure-python libraries.

##### An HPKE Python implementation:
https://github.com/dajiaji/pyhpke

This HPKE library is intended to be used by test code running on wptserve
that may receive an HPKE encrypted message and have to decrypt it and encrypt
a response.  This library is MIT licensed.

##### A CBOR Python implementation:
https://github.com/agronholm/cbor2

This CBOR library is intended to be used by test code running on wptserve that
may receive an CBOR encoded message and have to decode it and encode a response.
This library is MIT licensed.

### We're proposing adding these JavaScript libraries to a third_party/ subdirectory under our spec directory as per [this advice](https://github.com/web-platform-tests/rfcs/issues/46#issuecomment-587707539):

These HPKE and CBOR libraries are used by test code to decrypt and decode data
coming from JavaScript APIs to verify their contents, and used by test code to
encode and encrypt response data.  These libraries and all of their included
dependencies are MIT licensed.

##### An HPKE JavaScript implementation:
https://github.com/dajiaji/hpke-js

This library has some dependencies, so we're proposing building/transpiling into a single hpke.js file.
This library is MIT licensed.

##### A CBOR JavaScript implementation:
https://github.com/paroga/cbor-js/blob/master/cbor.js

This library is MIT licensed.

##### An Ed25519 JavaScript implementation:
https://github.com/paulmillr/noble-ed25519

This Ed25519 library is intended to be used by JavaScript test code
that may receive an Ed25519 private key and message to sign that message, or
a public key and signature to verify that signature.  This library is MIT licensed.

## Risks

Users of these libraries may need to update them from time to time if new
functionality or fixes are required. This is likely not a big risk.

The HPKE library, once built/transpiled into one JavaScript file, may be slightly harder to debug, but we'll disable minification when building.
