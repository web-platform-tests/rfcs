# RFC 215: Defining the Scope of web-platform-tests

### Summary

Define the scope of web-platform-tests to specifications which might reasonably be called "web specifications"
and which are intended to be implemented by web browsers.

Additionally, define what "tentative" might reasonably be used for.

This is intended to codify existing practice,
rather than represent any change.


### Details

Historically, what's been in scope for web-platform-tests has been somewhat nebulous,
and largely based on a common understanding of participants,
especially those on the Core Team.
We should put this on surer footing,
and make it easier for others to understand what is and isn't reasonable to add.

The primary aim of web-platform-tests is to test
[web specifications](https://github.com/w3c/browser-specs/blob/11a71b738f5e41f9239fdcd2074153388c8c6b8b/README.md#spec-selection-criteria)
[intended to be implemented by browsers](https://github.com/w3c/browser-specs/blob/11a71b738f5e41f9239fdcd2074153388c8c6b8b/README.md#categories),
both RFC 2119 "must" and RFC 2119 "should"
implementation requirements.

A specification does not need to have cross-vendor support for its tests to be included in web-platform-tests.
(XXX: Should we set a bar of one-vendor support or are we okay with zero-vendor support?)

Additionally, other features can be added as
[tentative](https://web-platform-tests.org/writing-tests/file-names.html#:~:text=.tentative,-%3A%20)
tests:

 * Web browser behavior currently being explored via an
   [explainer](https://tag.w3.org/explainers/),
   but without a specification yet written.
   (XXX: require prototyping to have started?)

 * Historic but unspecified features in web browsers,
   especially where major browsers are interoperable,
   where there exists a long-term intention to specify them.
   (Note: this excludes features which have been deliberately removed from specifications.
    These are explicitly out of scope.)

Features which are optional 
(in RFC 2119 terms:
 where the conformance criteria is
 "may" or "may not")
may be included as
[optional tests](https://web-platform-tests.org/writing-tests/file-names.html#:~:text=.optional,-%3A%20)
or
[optional subtests](https://web-platform-tests.org/writing-tests/testharness-api.html#optional-features).


### Risks

We over-constrain what is allowed in web-platform-tests,
potentially raising the bar to change what is allowed to "submit a new RFC",
leading browser vendors to reduce what they submit to the project.
