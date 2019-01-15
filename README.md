**This process is not yet in effect, please see [issue #1](https://github.com/web-platform-tests/rfcs/issues/1).**

# web-platform-tests RFCs

Most changes in web-platform-tests can be made directly in pull requests
following the usual review process.

This RFC (request for comments) process is used to request wider review.

## When to use the RFC process

Use this process for substantial changes which might impact other stakeholders
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

As a rule of thumb, please use the RFC process when it would be useful and err
on the side of skipping it if in doubt!

## The RFC process

 - Create a [new issue](https://github.com/web-platform-tests/rfcs/issues/new)
   in this repository by filling out the issue template.
 - The proposal is discussed by the community and it is assumed that the
   proposal will change in accordance with that discussion. 
 - In the case of no substantive disagreement the RFC is considered accepted
   after 1 week.
 - If substantive disagreement remains, then the issue is escalated to the
   [core team](https://github.com/orgs/web-platform-tests/teams/wpt-core-team/)
   for a decison. Only arguments that have already been made on the issue may
   be taken into account. As a last resort, the core team may decide using a
   2/3 majority vote. A decision should be made within 1 additional week.
 - Close the RFC issue when it has been implemented.
