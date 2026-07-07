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
  - [Chromium AI Coding Policy](https://chromium.googlesource.com/chromium/src/+/4f44016dfbd4fcd890694c00d7f9ec6dcefe4955/agents/ai_policy.md)
  - [Firefox AI Coding Policy](https://github.com/mozilla-firefox/firefox/blob/1f7030c8de8f2b349c7d91d7b5a3253c109a1cc1/docs/contributing/ai-coding.md)
- prohibitive
  - [Code of Conduct ⚡ Zig Programming Language](https://ziglang.org/code-of-conduct/#strict-no-llm-no-ai-policy)
  - [Getting Started - The Servo Book](https://book.servo.org/contributing/getting-started.html#ai-contributions)

## Details

The following text describes the policy in full and will be maintained in a
dedicated document within WPT's `docs/writing-tests/` directory (which will be
referenced both from the project's `README.md` file and the
`docs/writing-tests/index.md` file):

> ### Guidelines for acceptable LLM use
>
> The use of large language models (LLMs) as tools to help author commits to
> this repository is allowed with the stipulations described below.
> Contributors who repeatedly fail to adhere to these guidelines may be banned
> from contributing to this project.
>
> #### Disclosure
>
> If LLMs are used as a significant input to a commit, authors are encouraged
> to include details about how they were used as part of the commit message in
> order to help review and future understanding of the code.
>
> Human-authored code discourse (e.g. issue descriptions, pull request
> descriptions, and responses to discussion threads) should not include
> LLM-generated content in the main text; any such content must be clearly
> labelled and placed inside a `<details>` element.
>
> #### Attribution
>
> All commits must be attributed to the human who is taking responsibility for
> them, regardless of LLM use.
>
> #### Understanding
>
> Every pull request must be initiated by one human. That person must author
> the pull request description, understand every change proposed, and be
> prepared to engage in technical discussion regarding those changes.

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
take shortcuts which undermine the quality of contributions. Any permissive
policy might be taken as encouragement to rely on fallible tools (LLMs are
particularly susceptible to certain kinds of test-writing errors, such as
over-fitting and fabrication).

However, it will not be possible to strictly enforce any policy. It inevitably
falls on contributors to follow rules and for administrators to police
transgressions. Respect in public works projects is never guaranteed; policies
exist only to make expectations clear (this is the same dynamic that guides the
design and enforcement of codes of conduct).
