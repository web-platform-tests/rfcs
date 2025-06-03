# RFC 224: Remove implicit session creation for webdriver client

## Summary

Trying to run WebDriver commands shouldn't implicitly create a new session,
because doing so leads to surprising behaviour.

## Background

Going all the way back to the [original implementation](https://github.com/w3c/wptrunner/pull/109) of our WebDriver client,
it has implicitly created sessions when commands are run if there isn't a current session.

With wdspec tests,
we can end up the teardown of fixtures creating a new session just to run their teardown steps,
which almost invariably don't apply to a new session.
Creating a session can also be somewhat slow,
so there can be a cost associated with this too.

## Details

We should simply remove the implicit session creation.

This already has an implementation PR,
[#51868](https://github.com/web-platform-tests/wpt/pull/51868).

This doesn't require any changes to wdspec tests,
as the fixtures already guarantee each test starts with a session.

It does require changing the resources tests,
which do currently rely on this behaviour.

## Risks

### Vendor-specific wdspec tests

While the fixtures in `webdriver/tests/support/` guarantee that a valid session exists,
vendor-specific wdspec tests could rely on the implicit session creation.

### Other usage of webdriver

While we've never actually published it as a library,
it's plausible that there exists other usage of `webdriver` outside of WPT.
This may break any such usage.
