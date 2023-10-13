# RFC XXX: add a `add_top_level_completion_handler` to testharness.js

## Summary

Make it possible to attach a completion handler that only runs if no
`RemoteContext` is listening for results from a `Tests` object.

## Details

Currently, if you have a completion handler which outputs to `console.log` (or
something else similarly global), then you'll get output from both the top-level
test (i.e., the document loaded by the test URL) but also from any other window
opened or any nested frames.

This is fundamentally the cause of
https://bugs.webkit.org/show_bug.cgi?id=259409, as we're considering the test as
having to run to completion when a completion handler first runs (and it seems
to be somewhat nondeterministic which order they run in, but that just adds
flakiness to the already potentially bad race).

This RFC proposes adding an `add_top_level_completion_handler` API to
testharness.js which will only run if no `RemoteContext` is listening for a
`Tests` results.

Specifically, this requires modifying `RemoteContext` to send a message that
causes the remote context to save the "is remote context" state, to then be able
to avoid running the top-level completion handler.

Currently, this technically isn't needed: a completion handler can simply guard
itself with `if (window !== window.top || window.opener !== null) return;`,
however if at some point we make it possible to run `fetch_tests_from_window` on
a window opened with `noopener` this then becomes a necessity.

## Risks

This is a simple API addition to `testharness.js` that allows
`testharnessreport.js` to ensure they only get completion handlers they care
about; this should pose only the risks associated with any modification of
`testharness.js`.
