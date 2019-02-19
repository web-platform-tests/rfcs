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
 - In the case of no substantive disagreement the RFC is considered accepted
   after 1 week. If any participant requests it, the comment period is extended
   to 2 weeks.
 - If substantive disagreement remains, then the issue is escalated to the
   [core team](https://github.com/orgs/web-platform-tests/teams/wpt-core-team/)
   for a decison:
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
