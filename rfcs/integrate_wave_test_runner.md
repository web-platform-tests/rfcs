# Integration of the Web Media API Test Suite 2018 into WPT

## Summary

This RFC discusses the integration of the Web Media API Test Suite 2018 which is based on Web-platform-tests.  

## Details

The [Web Media API Test Suite 2018 (WMATS2018)](https://github.com/cta-wave/WMAS) is a test suite for the [Web Media API Snapshot 2018](https://www.w3.org/2018/12/webmediaapi.html) specification. The test suite and specification are being developed as part of
the [CTA WAVE Project](http://cta.tech/WAVE). This project is forked from [W3C Web Platform Tests](https://github.com/web-platform-tests/wpt) and is customized
to automate test runs on web browsers for embedded devices and appliances suchs as TV sets, set-top boxes, consoles, etc. A hosted version is available at: https://webapitests2018.ctawave.org
This RFC is created as a recommendation to [this WPT issues](https://github.com/web-platform-tests/wpt/issues/16214) where the initial discussion to integrate the Web Media API Test Suite into WPT is started. Please refer to the comments in this issue to get more insights in the discussion. 

## Risks

The new test runner incoporates components that run on the client (DUT and Companion device) and on the server. The server components are written in JavaScript and run in Node.js, which could be a risk for WPT since a new depencencie (Node.js) is introduced. If this is considered as an issue, it remains the option to port the JavaScript code to Python, but this is associated with additional effort. 

## UPDATE
The implementation of Node.js was ported to Python ([CTA](https://cta.tech/) contracted [Fraunhofer FOKUS](https://www.fokus.fraunhofer.de/fame) for this purpose) and the [WPT Pull Request](https://github.com/web-platform-tests/wpt/pull/21323) was created, reviewed and merged. To minimize the burden on WPT tooling, all steps were taken to isolate the runner from the rest of the WPT stack. Integration tests were also added. The [CTA WAVE project](https://github.com/cta-wave/) will maintain the new test runner in the future and may contract third parties for this purpose. John Riviello (@JohnRiv), chair of the CTA WAVE HTML5 API Task Force, is the contact person for further questions.
