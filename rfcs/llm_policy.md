# RFC 239: Policy on LLM assistance in contributions

## Summary

Introduce guidelines for acceptable use of large-language models when
contributing to web-platform-tests.

## Background

[#202 Set policy for LLM-generated
tests](https://github.com/web-platform-tests/rfcs/issues/202) includes evidence
for public interest in a formal policy for LLM usage in authoring contributions
to WPT.

The Chrome team is exploring applications of LLMs for detecting coverage gaps
and for filling those gaps with generated code. ([Project
repository](https://github.com/GoogleChromeLabs/wpt-gen), [April 2026
presentation](https://www.youtube.com/watch?v=9r0PBbJFLoM))

A few examples of policies on LLM use in FOSS contributions:

- permissive
  - [ghostty/AI_POLICY.md at main · ghostty-org/ghostty](https://github.com/ghostty-org/ghostty/blob/main/AI_POLICY.md)
  - [Policy about LLM generated code from PRs · Issue #28335 · opencv/opencv](https://github.com/opencv/opencv/issues/28335)
  - [CONTRIBUTING.md: Guidelines relevant to AI-assisted contributions by gasche · Pull Request #14052 · ocaml/ocaml](https://github.com/ocaml/ocaml/pull/14052)
  - [LLVM AI Tool Use Policy — LLVM 23.0.0git documentation](https://llvm.org/docs/AIToolPolicy.html)
- prohibitive
  - [Code of Conduct ⚡ Zig Programming Language](https://ziglang.org/code-of-conduct/#strict-no-llm-no-ai-policy)
  - [Getting Started - The Servo Book](https://book.servo.org/contributing/getting-started.html#ai-contributions)

## Details

Proposed text:

> ### For Individual Contributors
>
> #### Disclosure
>
> Contributions that contain substantial amounts of tool-generated content must
> be labeled as such.
>
> #### Attribution
>
> Commits generated entirely by an LLM must be attributed to the LLM in the
> "Author" field.
>
> #### Understanding
>
> Every pull request must be initiated by one human. That person must author
> the pull request description, understand every change proposed, and be
> prepared to engage in technical discussion regarding those changes.
>
> ### For Trusted External Review
>
> Some external projects conduct review which the WPT maintainers recognize as
> authoritative. From rendering engines like Gecko to dedicated test suites
> like WASM, patches merged in these projects are incorporated into WPT without
> further review. The policy outlined by this document does not apply to these
> contributions; the external projects are trusted to determine their own
> mechanisms for quality assurance.

## Risks

### Discouraging volunteers

All but the most permissive policy is effectively another hurdle to
contributing to the project. Friction in the contribution process could deter
people who might otherwise volunteer their time to help improve the project.

In some sense, adding friction is the goal of this policy. New technology has
removed barriers which previously restricted unqualified individuals from
participation. Rather than introducing more restrictions on good-faith actors,
an ideal policy will buttress eroded structural barriers with more intentional
social ones.

### Encouraging low-value contributions

All but the most restrictive policy could be interpreted as an invitation to
take shortcuts which undermine the quality of contributions.

However, it will not be possible to strictly enforce any policy. It inevitably
falls on contributors to follow rules and for administrators to police
transgressions. Respect in public works projects is never guaranteed; policies
exist only to make expectations clear (this is the same dynamic that guides the
design and enforcement of codes of conduct).
