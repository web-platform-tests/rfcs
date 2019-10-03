# RFC 32: Asynchronous setup

## Summary

Extend testharness.js with a new API that makes it easier for test authors to
defer subtest execution until the completion of some asynchronous operation.

## Details

Many testharness.js tests in WPT need to perform some asynchronous operation
before subtests may be executed. For instance:

- WebIDL tests need to fetch data (IDL files)
- CSS tests need to wait for fonts to be ready
- Service worker tests need to wait for worker registration

Test authors can achieve this today (that is, without any extension to
testharness.js) through three different approaches:

1. Explicitly waiting for the setup operation to complete from every subtest
2. Performing the setup operation in a dedicated `promise_test` which is
   defined before all others
3. Enabling the `explicit_done` option, perform the setup operation, define the
   tests, and finally invoke the global `done` function

However, each of these approaches has subtly different semantics and each
requires additional code which obscures the intent of the test and increases
the probability of bugs.

A new API would improve consistency of results, reduce boilerplate, and reduce
the associated risk of subtle errors.

Introduce a new global function named `promise_setup` which implements the same
signature as the existing global function named `setup`. This function should:

1. If no function is provided:
   1. Set the harness status to `ERROR`
   2. Transition the harness to the `COMPLETE` phase
   3. Abort this algorithm
2. Apply any harness configuration parameters provided
3. Invoke the provided function and capture the result
4. If the previous step produces an exception or if the result is not a
   "thenable" object:
   1. Set the harness status to `ERROR`
   2. Transition the harness to the `COMPLETE` phase
   3. Abort this algorithm
5. Configure testharness.js to not execute any tests which are subsequently
   defined
6. Wait for the result of the provided function to settle or for the amount of
   time equal to the internal harness timeout to pass (whichever comes first)
7. If the result of the provided function has not settled:
   1. Set the harness status to `TIMEOUT`
   2. Transition the harness to the `COMPLETE` phase
   3. Abort this algorithm
8. If the result of the provided function is rejected:
   1. Set the harness status to `ERROR`
   2. Transition the harness to the `COMPLETE` phase
   3. Abort this algorithm
9. Configure testharness.js to execute any tests which are subsequently defined
10. Begin execution of any tests which were defined while this algorithm waited

These steps are intended to approximate the corresponding synchronous behavior
of the `setup` function.

## Risks

Increasing the size of the API makes the harness more difficult to learn, and
this may discourage new contributors.

## Alternatives considered

Overload `setup` - extend the existing `setup` function to implement the
algorithm described above only if the provided function returns a "thenable"
object (and to preserve its existing behavior otherwise). This style of API is
difficult to discover and error prone.
