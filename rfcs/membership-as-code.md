# RFC #25: Maintain Membership in Code

## Summary

Introduce [a new "membership"
repository](https://github.com/web-platform-tests/membership) which stores the
list of administrators for [the web-platform-tests GitHub
organization](https://github.com/web-platform-tests), and use open source
tooling to synchronize that list with the state of GitHub.com.

## Details

https://github.com/web-platform-tests/wpt/issues/11548 requests a public
listing of the WPT administrators. In
https://github.com/web-platform-tests/wpt/pull/10832, we published this
information as a static list in the project documentation. That list quickly
went out of date.

[Terraform](https://terraform.io) is an open-source and cross-platform tool
designed to maintain service configuration in code. By using it to manage the
organization's GitHub settings, we can offer a public membership list that is
always up-to-date. By storing the list in a git repository, we can provide a
record of how membership status has changed over time.

This approach also increases transparency in the process of joining and leaving
the organization: such changes can be reviewed, discussed, accepted, and
rejected in the same manner as more traditional software-based pull requests.

## Risks

Administrators may mistakenly subvert this process and make membership changes
using the GitHub.com user interface. Removals will be detected by the next
invocation of Terraform, and the operator can update the repository as
necessary. Additions will go unnoticed and detract from the goal of
transparency.

This may not be appropriate for maintaining membership status of
non-administrators. As of 2019-08-07, 385 people have a non-administrative
membership, but very few have publicized this information. This solution does
not support private membership.

Terraform is not currently used anywhere within the WPT infrastructure.
Introducing it here will increase the learning curve to becoming an
administrator.
