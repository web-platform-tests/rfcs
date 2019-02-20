# RFC 13

## Summary
Add testdriver.js support for the `generate_test_report` WebDriver command, which will allow generic reports to be consistently generated during web platform tests. This command is specced as [part of the Reporting API spec](https://w3c.github.io/reporting/#generate-test-report-command).

## Details
The real-world causes of reports (like deprecations, interventions, and crashes) can differ between different user agents. In order to test reporting consistently, we need the ability to generate reports universally across the web platform. This command provides that ability.

## Risks
The WebDriver endpoint is new and may still require changes, which would then affect the testdriver.js API as well.
