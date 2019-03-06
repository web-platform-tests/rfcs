# RFC #15: Reftest graph simplification

## Summary

Simplify what we allow within the reftest graph.

## Details

### Background

Currently, reftests form a graph which can quickly end up with a large number of pass conditions.

In rough principle, we load everything with `<link rel=match>`/`<link rel=mismatch>` into a directed graph (with URLs as vertices, `match`/`mismatch` as edges), and then make every source node (i.e., a node with no incoming edges) into a "test".

From each test, we enumerate all possible walks from the source node with either no repeating nodes and ending at a sink node (i.e., a node with no outgoing edges), or ending at the first repeated node.

In reality, we have very little complexity within the graph, as of [37e794f690](https://github.com/web-platform-tests/wpt/tree/37e794f69091ffed1dfa7399a3ef154125fddda9): we have 14954 tests, and only 219 are more complex than a simple two page comparison.

Of those 219:
 * 111 just have alternates at the top level (the basic OR case),
 * 55 are trails with no repeating vertices (the basic AND case),
 * 48 are trails ending in a cycle (still a basic AND),
 * leaving us with 5 more complex cases (which mix AND and OR).
 
These five more complex cases are given [here](https://gist.github.com/gsnedders/7874fe11acd1dc8eaacb7448db8ca690). All but one of these are `css-transforms` that have been known to be broken for years and are quite clearly bogus (many pass conditions are subsets of one another!). The only other example is `/css/CSS2/text/text-indent-wrap-001.xht`, which must be \[(not) equal to `/css/CSS2/text/text-indent-wrap-001-notref-block-margin.xht`] OR \[equal to `/css/CSS2/text/text-indent-wrap-001-ref-inline-margin.xht` AND `/css/CSS2/text/text-indent-wrap-001-ref-float.xht`]. Looking at the individual tests, it appears that all of these are mistaking multiple links as conjunctions, and all of these should be simple chains rather than alternates; [#15523](https://github.com/web-platform-tests/wpt/pull/15523) does this. (This also makes me suspicious as to how many of the 111 tests we have with alternates make a similar mistake, and whether we can make this clearer.)

We also have a number of "tests" which aren't actually found in the manifest, because we have a cycle of "tests" (and therefore we have no source node among them); these are given [here](https://gist.github.com/gsnedders/ce17e7df69ee89ce471d5a553b826dd2). Note that the majority of these are tracked in [#5294](https://github.com/web-platform-tests/wpt/issues/5492) and fixed in [#15523](https://github.com/web-platform-tests/wpt/pull/15523), and the rest are broken `/infrastructure` tests that are meant to test for support of cycles.

### Proposal

This RFC contains a number of proposals, given below:

#### Test Detection

With the current logic, we need to process every changed file in the repository before running a single given path, as we need to recompute the reftest graph to determine whether that path is still a source node in the graph. This current behaviour also sometimes causes confusion from users when files they believe to be tests do not appear in the results.

While purely judging whether or not something is a test from the path (presumably with [`not name_is_reference`](https://github.com/web-platform-tests/wpt/blob/6111e21ca42dea24a07df1acfbed78769a415b5b/tools/manifest/sourcefile.py#L335)) will lead to redundant tests (e.g, `test-001.html == test-002.html == test-ref.html` will now lead to two tests), this matches user expectations more clearly and makes it possible to do partial manifest updates for reftests.

#### Valid Graphs

It becomes much easier to explain why a reftest fails if we have less complexity in what is allowed within the graph. Given we have so few complex cases, it seems like we should be able to massively simplify what we allow.

In short, the proposal is that from a test node:

 * If the out degree of the test node is > 1, then each reachable node from the test node must have an out degree of 0. (i.e., if we have alternates, no reference can have a `<link rel=match>`)
 * If the out degree of the test node is 1, then every reachable node from the test node must have an out degree of ≤ 1. (i.e., if we have conjunctions, it's a simple chain with no alternation)
 
## Advantages

* Simplify runner implementations,
* Makes it simpler to explain failure to users.

## Disadvantages

* Unable to deal with more complex cases that seemingly never appear in the wild but have been occasionally stated that we need.
* Linting graph structure means following references.
