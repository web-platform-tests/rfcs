## RFC #18: Simplify License

### Summary

Reduce WPT's license to the 3-clause BSD license and update the copyright
holder to "web-platform-tests contributors."

### Details

WPT is currently dual-licensed under [the W3C Test Suite
License](#w3c-test-suite-license) and [the W3C 3-clause BSD
License](#w3c-3-clause-bsd-license). Both licenses name "W3C" as the copyright
holder.

In [WPT issue gh-11009](https://github.com/web-platform-tests/wpt/issues/11009)
Philippe Le Hegaret (W3C) led a discussion about how this dual license is no
longer appropriate for the web-platform-tests project. Wendy Seltzer (W3C)
proposed changes to the Patents and Standards Interest Group (PSIG) and
reported:

> Per conversation with [Philip Jägenstedt (Google)] I'm filing two pull
> requests against license.md, one dropping from the dual license to the W3C
> 3-clause BSD; the other replacing with the straight 3-clause BSD (which
> differs only in not having the named copyright holder W3C). PSIG has given
> non-objection to the W3C 3-clause BSD. If that would still cause legal
> hiccups, then we can probably persuade them of the vanilla 3-clause.

Because [Philippe Le Hegaret and Philip Jägenstedt have made it known that
naming "W3C" as the copyright holder is
undesirable](https://github.com/web-platform-tests/wpt/pull/11191), we choose
to adopt the 3-clause BSD license which lists the copyright holder as
"web-platform-tests contributors."

### Risks

Pending the endorsement from the W3C Patents and Standards Interest Group,
there appears to be no risk associated with this change.
