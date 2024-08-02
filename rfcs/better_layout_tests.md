# RFC #205: Layout tests for non-browsers

## Summary

Make layout tests, particularly *modern layout* tests (`css-flexbox`, `css-grid`, etc) more accessible to non-browsers by reducing the capabilities they rely on to run.


## Background

### Non-browser layout engines

Libraries like [Yoga](https://github.com/facebook/yoga) (used by React Native) and [Taffy](https://github.com/DioxusLabs/taffy) implement CSS layout algorithms (such as Flexbox, CSS Grid, Block layout) which they wish to test for spec compliance, but they are not browsers and therefore do not have many of the capabilities that WPT layout tests assume. Capabilities that such libraries may not have available include:

- JavaScript execution
- Style resolution
- Visual rendering
- Legacy layout capabilities such as `float`

Additionally, while browsers do have these capabilities running tests that utilise them (via the WPT runner) can be slow. For example running just `css-flexbox` in Servo takes around 100 seconds. This can be contrasted with Yoga and Taffy's existing test suites which can run ~1000 tests in <5 seconds. As such, complete browsers may also be interested in improved layout tests that they could run more quickly (and independently of the wider browser), which would shorten iteration times when working on layout code.

### Existing Layout Test Types

Layout tests broadly fall into one the following categories \[have I missed any?\]:

- **Ref tests**: tests which are validated by visually rendering some HTML and comparing the result the visual rendering of some reference HTML (which is assumed to be simpler to render, often using hardcoded sizes rather than relying on layout algorithm features)

- **Simple checkLayout tests**: tests which use the `checkLayout` helper defined in [`resources/check-layout-th.js`](https://wpt.live/resources/check-layout-th.js) and call it exactly once. The argument to the `checkLayout` function is a string containing a CSS selector which selects elements, each of which forms a subtest.

- **Complex checkLayout tests**: tests which use the `checkLayout` helper defined in [`resources/check-layout-th.js`](https://wpt.live/resources/check-layout-th.js) and call it multiple times, executing scripts to modify the DOM in between. The arguments to the `checkLayout` function are a string containing a CSS selector which selects elements, each of which forms a subtest, and a boolean indicating whether this is the last time `checkLayout` will be called.

- **Other**: This includes tests that run custom scripts to test the layout, tests that specifically test updates to the tree, etc.

We will focus on the first two categories in this RFC.

## Problems with existing test types

**Ref tests:**

- They require a visual renderer which is not always available
- Visually rendering and then diffing renders is slow
- They are not very precise: They only test the overall visual result not the precise box dimensions so a test may pass incidentally.
- Some ref tests (although relatively few) rely on things that are complex to render such as dashed borders

**Simple checkLayout tests:**

- They require JavaScript execution which is not always available
- They sometimes rely on non-standard behaviour of `offsetTop`/`offsetLeft` for assertions

## Proposals

### Proposal 1: Layout ref tests

Add a new types of test, the "layout ref" test. These would be like a regular ref tests except that rather than compare a visual render to that of a reference snippet of HTML, they would compare the computed x/y/width/height/padding/border/margin values.

Additionally, rather than comparing the entire HTML document (which causes tests to be seperated), "layout ref" tests would mark the HTML elements that should be tested with a `data-test-root` attribute. Only subtrees of a `data-test-root` element would be checked. And each `data-test-root` would become a subtest.

Convert regular ref tests to "layout ref" tests where appropriate.

In some cases an implementation that is known to correctly pass a given test could be used to generate detailed assertions for a "layout ref" version of that test.

#### Advantages

- Does not require a visual renderer
- Diffs will be faster than using a visual renderer
- Will check the layout more precisely than visual render

#### Disadvantages

- Existing tests will need to be converted manually
- Doesn't test the visual rendering.

#### Alternatives

- Ref tests could be converted to "simple checkLayout" tests


### Proposal 2: Create a test transformation tool

Create a test transformation tool to statically translate "simple checkLayout" tests (and "layout ref" tests if proposal 1 is accepted) into a simple machine-parsable representation of both the HTML/CSS tree to test and the assertions to check.

The transformation should:

- Resolve and inline styles (as the intention here is to test layout not css resolution)
- Parse out the selector from the `checkLayout` call, resolve the elements it matches, and use that to mark tests roots and set subtest names in output.
- Output a standard format such as XML or JSON that can be easily parsed.

This machine-parsable representation could then be used to generate unit-style tests which directly call a layout engine using it's native API. Both Yoga and Taffy already have test generation setups for their own test suites.


## Alternatives

- Create a "test runner" that does have the capabilities like scripting, style resolution and visual rendering that standalone layout engines can be hooked into.

   This is seen to be a larger undertaking than simplifying and statically transforming tests. And will likely not come with performance benefits.