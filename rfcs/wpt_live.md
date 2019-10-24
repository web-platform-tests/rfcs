# RFC 36: wpt.live

## Summary

Maintain a publicly-accessible deployment of the web-platform-tests project,
hosted at http://wpt.live. Maintain another deployment which allows project
contributors to preview their submissions during the peer review process,
hosted at http://wptpr.live.

## Details

Bocoup and Google have collaborated on a service to provide a
publicly-accessible instance of the web-platform-tests. The source code and the
infrastructure configuration are maintained in the following repository:

https://github.com/bocoup/web-platform-tests.live

This system involves two services, accessible from two distinct hosts.

[wpt.live](http://wpt.live) runs the WPT project's "wptserve." It automatically
synchronizes with the latest revision of the `master` branch of the WPT git
repository. It is expected to be referenced by many highly-visible domains of
the web platform (e.g. web specifications, [wpt.fyi](https://wpt.fyi), and
[web-platform-tests.org](https://web-platform-tests.org)).

[wptpr.live](http://wptpr.live) also runs the WPT project's "wptserve," and it
includes additional logic to automatically retrieve patches submitted to the
WPT repository through GitHub.com. These submissions are made available for
"trusted" patches (i.e. those authored by project collaborators and those that
have been explicitly "safelisted" by project collaborators) in order to
facilitate peer review. The WPT GitHub project will be configured to inform
contributors of the status of these previews.

## Risks

A publicly-accessible service like wpt.live is a substantial responsibility for
the group of people maintaining it. Because it is expected to be referenced
from many independent projects, consistent availability is critical. Ensuring
that availability requires a different mode of support than much of the WPT
staff's existing responsibilities. Accepting and promoting a project like
wpt.live therefore involves the risk of unpredictable maintenance requirements.

The stability of wptserve is not guaranteed. It is derived from software whose
maintainers [explicitly discourage its use in production
contexts](https://docs.python.org/2/library/simplehttpserver.html).
Additionally, it is designed to execute code written by WPT test authors, and
these contributors may not have stability or security in mind.

wpt.live has been designed to mitigate this risk via multiple automated failure
recovery mechanisms. Instances use [Docker's restart
policies](https://docs.docker.com/engine/reference/commandline/run/) to rapidly
recover from transient runtime errors (e.g. memory leaks and stack overflows).
In addition, the project has been configured to use [Google Cloud Platform's
"autohealing"
feature](https://cloud.google.com/compute/docs/instance-groups/#autohealing) in
order to guard against more persistent problems (e.g. faulty disk state).

## Alternatives considered

Do nothing - the service described in this RFC is already provided by the W3C
via http://w3c-test.org. WPT could avoid the risks and continue to enjoy the
benefits by continuing to rely on that service. However, the service lacks
automated failure recovery mechanisms, and its unreliability is well-known by
both its users and its maintainers. Its status as a W3C-owned project reduces
the ability of WPT members to understand and fix problems.

Inherit the existing solution - To the latter point, another alternative is for
WPT to inherit w3c-test.org. However, because this would not improve
reliability, doing so would almost increase the maintenance effort required
from WPT project members. (If this proposal is accepted, the maintainers of
w3c-test.org will configure that domain to redirect to wpt.live).
