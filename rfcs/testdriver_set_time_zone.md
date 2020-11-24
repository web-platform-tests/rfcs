# RFC 73 : Add testdriver.js support for the `timeZone` WebDriver command.

## Summary
Add testdriver.js support for the `timeZone` WebDriver extension, which will
allow setting the current time zone in the browser. This extension is specced
as part of the [HTML spec](https://github.com/whatwg/html/pull/3047).

## Details
Changing the time zone is not available via normal web APIs. To test the effect
of changing time zone on the `timezonechange` event there needs to be a way
to instruct the browser to act as if the time zone is changing: this WebDriver
extension provides that ability.

## Risks
* The WebDriver endpoint is new and may still require changes, which would then
  affect the testdriver.js API as well.
* The WebDriver endpoint is very specific and will likely only be used for a
  small set of tests, so the maintenance to value ratio may be low.
    * That said, testdriver.js is designed to make supporting a command
      low-maintenance, and to not require every executor to support every
      testdriver.js command.
