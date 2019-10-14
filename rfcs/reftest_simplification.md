# RFC #15: Reftest graph simplification

## Summary

Simplify what we allow within the reftest graph.

## Details

### Background

Currently, reftests form a graph which can quickly end up with a large number of pass conditions.

In rough principle, we load everything with `<link rel=match>`/`<link rel=mismatch>` into a directed graph (with URLs as vertices, `match`/`mismatch` as edges), and then make every source node (i.e., a node with no incoming edges) into a "test".

From each test, we enumerate all possible walks from the source node with either no repeating nodes and ending at a sink node (i.e., a node with no outgoing edges), or ending at the first repeated node. Each walk becomes a sequence of ANDs, and at least one walk must be satisfied for the test to pass.

In reality, we have very little complexity within the graph, as of [a12f37b6a0](https://github.com/web-platform-tests/wpt/tree/a12f37b6a04bdbde0902b0fc36b6a46f4782bd06): we have 15905 tests, and only 253 are more complex than a simple two page comparison.

Of those 253:
 * 118 just have alternates at the top level (the basic OR case),
 * 132 are just trails (the basic AND case),
 * leaving us with 3 more complex cases (which mix AND and OR).
 
However, notably our definition of how we define the pass condition differs from what the [CSS WG had documented for their testsuite](https://wiki.csswg.org/test/format#reference-links) originally defined (in 2011), which the WPT behaviour was originally meant to match.

This behaviour is that if there is at least one `<link rel=match>` at least one must match, and all `<link rel=mismatch>` must match.

Of the 118 tests we detect as OR under our current logic, but [85 of these](https://gist.github.com/gsnedders/2ee57070569e177d973a6736f7d278bb#file-only_or_mixed_eq_cond-txt) have a combination of both match and mismatch, which is likely a bug under our current logic, and looking at a mostly random subset of these, it appears that the intention was the CSS WG documented behaviour.

There are also [7 tests](https://gist.github.com/gsnedders/2ee57070569e177d973a6736f7d278bb#file-only_or_mismatch-txt) that have multiple mismatches; again it appears like the intention was the CSS WG documented behaviour. In both categories, all of these are within `/css`, and  most of these tests predate WPT's manifest generation, so were just following the documented CSS WG behaviour.

### Proposal

This RFC contains a number of proposals, given below:

It is proposed that we get rid of the references-of-references (i.e., the current AND case) as this is the root of a large amount of implementation complexity and the current behaviour of only source nodes being considered tests has confused a number of users. It also requires us to process every file in the repository to update the manifest before we can run a single reftest, which would be desirable to avoid.

In general, we shall make every file with at least one `<link rel=match>`/`<link rel=mismatch>` into a test; this will introduce new tests for all current references which currently chain onto further references.

Further, it is proposed that we change the combinatorial behaviour to the one originally defined by the CSS WG: if there is at least one `<link rel=match>`, at least one must match, and all `<link rel=mismatch>` must match.

### Potential future additions

Some people have raised concerns about the ability to continue to do more complex AND/OR combinations, we could in the future re-introduce support for this through some explicit grouping (into AND) via some attribute on the `link` elements.

The risk of punting this to be future work is that it makes some tests impossible to write in web-platform-tests in the meantime. However, it has been eight years since the CSS WG defined their format and four years since we started supporting arbitrary reftest chains and it appears nobody has written such a test. Either it turns out this functionality is unneeded or nobody knows about it.

## Advantages

* Avoids needing to create a whole graph to find/execute reftests,
* Avoids tests disappearing when they are referenced elsewhere in the graph.
* Makes it simpler to explain failure to users.

## Disadvantages

* Loses the chains of failure when a reference renders incorrectly.
