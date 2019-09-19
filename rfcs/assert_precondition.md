# RFC #16: 'PRECONDITION_FAILED' subtest result (status)

## Summary

Allow another distinct subtest result, different from `FAIL` or `MISSING` results,
by asserting a precondition requirement for a subtest, where the precondition
needs to be met for the test to actually produce any meaningful result.

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

Add subtest result of `PRECONDITION_FAILED` - a result for subtests which
assert an unmet precondition, and thus do not run the remainder of the test.

Add an `assert_precondition(condition, description)` function helper, which will conclude
with a `PRECONDITION_FAILED` result when `condition` is not truthy.

> __assert_precondition(condition, description)__
>
> Concludes the test with `PRECONDITION_FAILED` status if `condition` is not truthy.
> Used for skipping subtests will not produce a meaningful result without the condition,
> e.g. optional features that are not implemented by the user agent.

#### Example Usage

    promise_test(function(test) {
      return subtle.generateKey(algorithm, extractable, usages)
        .then(
          function(result) { ... },
          function(err) {
            var supported = !isUnsupported(err); // "Unsupported" case determined ad-hoc
            assert_precondition(supported, algorithm + ' not implemented');
            assert_unreached("Threw an unexpected error: " + err.toString());
          }
        });
    }

## Advantages

For spec-compliant implementation omission, it would allow distinction from other
outcome implications.

 - `FAIL` implies an incorrect implementation
 - `MISSING` implies a test was not run at all
 - `PRECONDITION_FAILED` implies a test was not applicable

It also allows for more expressive tests, allowing authors to be explicit about their
expectations.

## Disadvantages

 - An extra outcome to consider
   - Will involve viral changes to logging, and log consumers
 - Possible metrics skewing from the point that tests change to use this status
