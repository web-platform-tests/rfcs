# RFC 127: Add RFC Exemption for testdriver.js methods similar to WebDriver endpoints

## Summary

To expedite changes and reduce developer friction, the web-platform-tests
community should consider allowing a developer to bypass the RFC process if the
developer is only extending testdriver.js with a method similar to an existing
WebDriver endpoint.

## Details

During the 2022-10-04 [meeting](https://github.com/web-platform-tests/wpt-notes/blob/master/minutes/2022-10-04.md),
attendees discussed RFC [#121](https://github.com/web-platform-tests/rfcs/pull/121)
and similar changes to testdriver.js. Attendees decided that RFC 121 could land
automatically if the WebDriver changes landed first.

Scaling the learnings from that, this RFC proposes that the README should add a
RFC exemption for `extending testdriver.js with a method that closely matches a WebDriver endpoint`.

To make the reviewer aware of the type of change, PR author should add the
`testdriver.js` label to the PR.

## Risks

Low/minimal risks:
- A developer trying to extend testdriver.js with a method that does not match
  an existing WebDriver endpoint.
  - Mitigation:
    - The PR review process can catch this if the WebDriver endpoint already
      exists.