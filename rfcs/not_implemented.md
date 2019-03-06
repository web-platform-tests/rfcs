# RFC #16: 'NOT_IMPLEMENTED' subtest result (status)

## Summary

Allow a distinction between `FAIL` or `MISSING` result, and a deliberate,
spec-compliant lack of some particular implementation.

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

Add an `unimplemented(description)` function to `test` (the argument passed to `test_function`s),
for completing with a `NOT_IMPLEMENTED` result.

> __test.unimplemented(description)__
>
> Concludes the test with `NOT_IMPLEMENTED` status. Used for optional features that are not
> implemented by the user agent.

#### Example Usage

    promise_test(function(test) {
      return subtle.generateKey(algorithm, extractable, usages)
        .then(
          function(result) { ... },
          function(err) {
            if ('NotSupportedError' in self && err instanceof NotSupportedError) {
                assert_unimplemented(err);
            }
            assert_unreached("Threw an unexpected error: " + err.toString());
        });
    }

## Advantages

For spec-compliant implementation omission, it would allow distinction from other
outcome implications.

 - `FAIL` implies an incorrect implementation
   - Bundled with genuinely incorrect implementations
   - Inflates any metrics enumerating failures
 - `MISSING` implies a test was not run at all
   - Indistinguishable from infrastructure error / failure to run

It also allows for more expressive tests, allowing authors to be explicit about their
expectations.

## Disadvantages

 - An extra outcome to consider
   - Will involve viral changes to logging, and log consumers
 - Possible metrics skewing from the point that tests change to use this status
