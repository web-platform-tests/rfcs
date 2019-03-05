# RFC #16: 'NOT_IMPLEMENTED' subtest result (status)

## Summary

Allow a distinction between `MISSING` result, and a deliberate, spec-compliant
lack of some particular implementation.

## Details

### Background

Some specs, such as [WebCryptoAPI](https://www.w3.org/TR/WebCryptoAPI/#scope-algorithms),
focus on the discovery of the set of a user agent's implementations, but do not mandate
any *particular* algorithm is implemented. As a result, it is perfectly spec compliant to
omit any (or, even **all**!) implementations.

With respect to testing, there are two options for results when enumerating the
set of implementations:

  - Consider a lack of implementation a `FAIL`
  - Omit results for known non-implementations (`MISSING` on wpt.fyi)

The current behaviour of [some WebCryptoAPI tests](https://wpt.fyi/results/WebCryptoAPI/encrypt_decrypt/aes_ctr.https.worker.html) actually has a mixture - some tests
are missing, some are "failing".

### Proposal

Add `NOT_IMPLEMENTED` - a specific test result for known, valid, non-implementations.

## Advantages

For spec-compliant implementation omission, it would allow distinction from other
outcome implications.

 - `FAIL` implies an incorrect implementation
   - Bundled with genuinely incorrect implementations
   - Inflates any metrics enumerating failures
 - `MISSING` implies a test was not run at all
   - Indistinguishable from infrastructure error / failure to run

## Disadvantages

 - An extra outcome to consider
 - Possible metrics skewing from the point that tests change to use this status
