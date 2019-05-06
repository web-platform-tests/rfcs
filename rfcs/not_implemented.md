# RFC #16: 'SKIP' subtest result (status)

## Summary

Allow a distinction from `FAIL` or `MISSING` results by deliberately
skipping a subtest that will not produce a meaningful result, such as a 
spec-compliant lack of feature implementation.

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

Add subtest result of `SKIP` - a result for deliberately skipped subtests.

NOTE: `SKIP` is a test status emitted by the wpt-runner

Add an `skip(description)` function to `test` (the argument passed to `test_function`s),
for completing with a `SKIP` result.

> __test.skip(description)__
>
> Concludes the test with `SKIP` status. Used for skipping subtests will not produce
> a meaningful results, e.g. optional features that are not implemented by the user agent.

#### Example Usage

    promise_test(function(test) {
      return subtle.generateKey(algorithm, extractable, usages)
        .then(
          function(result) { ... },
          function(err) {
            if ('NotSupportedError' in self && err instanceof NotSupportedError) {
                test.skip(algorithm + ' not implemented');
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
