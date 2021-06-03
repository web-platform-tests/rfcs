# RFC 84: Move web-platform-tests Chat to Matrix

## Summary

Move the official location for synchronous discussions to Matrix, and
retire the `#testing` channel on irc.w3.org.

The proposed channel is [wpt:matrix.org](https://app.element.io/#/room/#wpt:matrix.org).

## Details

### Context

Since web-platform-tests has started it has primarily used IRC as a
means for (quasi) synchronous communications ("chat"). In particular
the `#testing` channel on irc.w3.org has been used to answer questions
from test authors and collaborate on infrastructure development and
maintenance.

IRC has historically been the go-to medium for web-platform related
chat, with W3C, TC39 and WHATWG all using IRC channels as the primary
communication mechanism for standards work, and browser engine
projects including Chromium, Gecko, Servo, and WebKit having IRC as a
well-supported option for synchronous communication. This made it
likely that web-platform-tests participants were already using IRC in
some capacity, so an IRC channel was an obvious choice for
web-platform-tests.

Recently however, the limitations of IRC have become more apparent:

* New contributors who have not grown up with IRC find it confusing
  and limited compared to other contemporary chat services. For
  example the fact that you only receive messages when connected to
  the channel, and there's no built-in logging/history, is a source of
  confusion to people who expect to be able to ask a question,
  disconnect, and reconnect later to see the answer. Although it's
  possible to work around this with the correct clients or external
  logs, this requires investment in understanding the IRC ecosystem
  that new users by definition don't have.

* The general problem with lack of moderation and difficulty
  controlling spam can make IRC an unwelcoming environment, especially
  for people in groups who frequently experience harassment
  online. These people may prefer to not participate in a project
  rather than use IRC.

As a result of this, and other factors, other projects have
increasingly moved away from IRC, and it is no longer the "default"
choice for web-platform work. In particular:

* Gecko moved to Matrix, hosted on
  [chat.mozilla.org](https://chat.mozilla.org).

* [WHATWG](https://app.element.io/#/room/#whatwg:matrix.org)
  [[discussion](https://github.com/whatwg/meta/issues/210)] and
  [TC39](https://app.element.io/#/room/#tc39-general:matrix.org) moved
  to Matrix, hosted on [matrix.org](https://matrix.org).

* [WebKit](https://webkit.org/getting-started/#contribute) moved to Slack.

* Chromium moved to an invite-only Slack instance.

* Servo moved to [Zulip](https://servo.zulipchat.com/)

* W3C is still extensively using IRC, but some groups have moved to
  [Slack](https://w3ccommunity.slack.com/).

This means that it can no longer be assumed that web-platform-tests
contributors are already using IRC as part of their participation in
web platform work. Along with the disadvantages of IRC compared to
modern alternatives, this suggests that we should also reconsider our
choice here.

### A Rather Biased Discussion of the Options

On the basis that having to use an additional chat system is itself a
barrier to contribution, we should look for alternatives that are
already used by the wpt-adjacent projects above, rather than choosing
from the large set of all existing systems. This suggests the two
reasonable alternatives are Matrix and Slack.

Slack's primary business is company-internal communications. This is
reflected in the fact that Slack instances typically require an
invite. Although it's possible to make instances that are broadly
accessible, the emphasis is very much on closed communities. This
provides a significant barrier to entry for new contributors, who may
feel they shouldn't look for an invite until they are already known
within a community. In addition the fact that Slack is a centralised
service run for profit risks the terms of use changing in an
unfavourable way in the future (e.g. to require payment).

Matrix is an open protocol with which it's possible to self-host a
server, and use a custom client. However, compared to IRC, there's a
clear default client (Element), which runs in a web browser and offers
a polished user experience. Whilst the feature-set of Matrix/Element
does not fully match that of Slack, it exceeds what's possible over
IRC, offering per-channel history, limited rich text in messages, link
previews, reaction emoji, etc. It also offers extensive moderation
tools which suggests it will address some of the concerns around
safety on IRC. Compared to Slack the focus is more on open
communities, and specific invites are not required to join public
channels.

### Proposal

We set up a matrix channel,
[wpt:matrix.org](https://app.element.io/#/room/#wpt:matrix.org),
hosted on matrix.org, and point to this as the official channel for
web-platform-tests chat, fully replacing IRC.

This matches TC39 and WHATWG. Although self-hosting would provide some
benefits in terms of not depending on third-party infrastructure, the
work required is not worthwhile in the short term. The federated
nature of Matrix should make it possible to move if hosting on
matrix.org ever becomes problematic.

### Admins

We should have multiple channel admins to ensure the channel can
continue operating smoothly in the absence of any
individual. Initially I propose the following, drawn from members of
the core team who are regularly on IRC (using GitHub usernames):

* @jgraham
* @foolip
* @gsnedders
* @sideshowbarker

It's unclear if there is a good reason to additionally have moderators.

### Documentation

All documentation pointing to the IRC channel will be updated to point
at the matrix channel instead.

### Risks

 * Bots which are present on IRC may not be available for Matrix. In
   particular we are using bitbot to notify about GitHub issues and
   PRs in the IRC channel.

   The use of bitbot is already somewhat controversial, with some
   participants disliking the frequent notifications. So it may not be
   a net loss to simply turn the bot off and require people to use
   another method to keep up with GitHub notifications. Alternatively
   we could keep bitbot running on IRC and allow people to use the
   Matrix/IRC bridge to see those messages, whilst otherwise not using
   the IRC channel. Given the popularity of GitHub, it is also likely
   we can find a Matrix bot to replace bitbot, which could then run in
   a separate wpt-notifications channel.

 * Changing communication mechanisms may fragment the community.

   This seems low risk; informally there was little dissent to the
   idea of moving to Matrix, and with other projects already making
   similar moves it seems more likely that contributors will
   participate on Matrix than on IRC. This would match the experience
   of Mozilla where the matrix channels quickly proved more active
   than IRC had ever been.
