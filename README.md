# web-platform-tests RFCs

Most changes in web-platform-tests can be made directly in pull requests
following the usual review process.

This RFC (request for comments) process is used to request wider review.

## When to use the RFC process

Use this process for substantial changes which would impact other stakeholders
or users of web-platform-tests. Examples of where it is likely to be useful:

 - Changes in [resources/](https://github.com/web-platform-tests/wpt/tree/master/resources)
   which would affect many test authors, such as extending testharness.js.
 - Changes in [tools/](https://github.com/web-platform-tests/wpt/tree/master/tools)
   which would affect downstream users of web-platform-tests, such as changing
   the manifest format or wptserve behavior.
 - Adding a new third-party code or service dependency.
 - Changing review workflows or policies.

Cases where the RFC process need *not* be used:

 - Introducing a new top-level directory for a new specification.
 - Extending testdriver.js with a method that closely matches a [WebDriver Classic](https://w3c.github.io/webdriver/) endpoint or a [WebDriver BiDi](https://w3c.github.io/webdriver-bidi) command or an event. To notify maintainers of testdriver.js vendor integration, label the pull request `testdriver.js`. (This exemption was introduced by [RFC 127](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/rfc_exemption_testdriver_method.md) and [RFC 185](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/testdriver_bidi.md).)
 - Minor changes in behavior in where all call sites are known and accounted
   for.
 - Behavior-preserving refactoring with a low risk of regressions.

## The RFC process

 - Each RFC requires a PR in this repository consisting of a single
   markdown file added to the `rfcs` directory setting out the proposed
   change. This file must have the PR number as the title and at least
   the following sections:
    * Summary
    * Details
    * Risks
 - The proposal is discussed by the community and it is assumed that the
   proposal will change in accordance with that discussion.
 - At least one review approval is required to merge a PR. For changes that
   may impact downstream WPT consumers there is an additional requirement
   of agreement from those consumers that the proposed change can be
   accommodated in their workflows.
 - Anyone is welcome to add the PR to the monthly
   [WPT infra meeting](https://github.com/web-platform-tests/wpt-notes) to
   have a dsicsussion about it. To add an RFC to the agenda, tag the PR with the
   `agenda+` label.
 - If reviewers or downstream consumers are not responsive after at least two
   weeks, the PR should be added to the monthly
   [WPT infra meeting](https://github.com/web-platform-tests/wpt-notes) to
   ensure it gets discussed.
 - After review approval(s) and in the case of no substantive disagreement, the
   RFC is considered accepted 1 week after the first approval. If any participant
   requests it, more time can be granted to review the proposal. This will not be
   less than one week but may be longer (e.g. until the next WPT infra meeting).
   However if no feedback is forthcoming by the agreed date, the proposal may
   be considered accepted.
 - If substantive disagreement remains, then the issue is escalated to the
   [core team](https://github.com/orgs/web-platform-tests/teams/wpt-core-team/)
   for a decision:
   - Only arguments that have already been made on the issue may be taken into
     account.
   - If meaningful new information is presented, the comment period can be
     extended to ensure that all participants have time to consider it.
   - When no new information is forthcoming, a decision should be made within 1
     additional week.
   - The decision can be to defer the issue until a later time.
   - If necessary, the core team may decide using a &geq;2/3 majority vote.
 - An RFC that is accepted is merged. One that is rejected is closed
   without merging.
