# RFC 209: Support testdriver.js in other test types

## Summary

Allow reftests, print-reftests, and crashtests to use `testdriver.js`, and add
the necessary support to `wpt {run,manifest,lint}`.

## Motivation

[`testdriver.js`][0] provides APIs that allow tests to send WebDriver-like
commands from client-side JavaScript.
Tests often use testdriver.js to access powerful capabilities that would
normally not be automatable because the web platform does not expose them.

Currently, only [testharness tests] may use testdriver.js.
However, test authors [have expressed interest] in extending testdriver.js
support to [reftests], WPT's main source of rendering coverage.
The features they wish to test with respect to rendering are often gated on
user activation or require pointer input, which can't be obtained without
testdriver.js or manual interaction:

| Tests | testdriver.js usage | Purpose |
| --- | --- | --- |
| [`fullscreen/`][1] | `bless()` | Provide [required transient activation][2] |
| [`customizable-select/`][3] | `bless()`, `click()` | Provide transient activation [required to show `<select>` pickers][4] |
| Various [`css/`][5] | `Actions().pointerMove()` | Exercise `:hover` styles |

* The `customizable-select/` tests are confined to [Chromium's
  `wpt_internal/`][6] because only Chromium's `run_web_tests.py` and
  [`testdriver-vendor.js`][7] can run them.
  These tests are effectively not portable to other vendors.
* The other tests have already landed in web-platform-tests/wpt, but
  [time out in all browsers][1] when run with [wptrunner], WPT's canonical test
  harness.
  These tests don't provide reliable coverage.

[have expressed interest]: https://github.com/web-platform-tests/wpt/issues/13183
[wptrunner]: https://web-platform-tests.org/tools/wptrunner/README.html
[testharness tests]: https://web-platform-tests.org/writing-tests/testharness.html
[reftests]: https://web-platform-tests.org/writing-tests/reftests.html
[0]: https://web-platform-tests.org/writing-tests/testdriver.html
[1]: https://wpt.fyi/results/fullscreen/rendering?run_id=6207808991395840&run_id=5099292293595136&run_id=5124412349349888&run_id=5184801065926656
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Element/requestFullscreen#security
[3]: https://chromium.googlesource.com/chromium/src/+/main/third_party/blink/web_tests/wpt_internal/html/semantics/forms/the-select-element/customizable-select/
[4]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLSelectElement/showPicker#security_considerations
[5]: https://github.com/search?q=repo%3Aweb-platform-tests%2Fwpt+path%3Acss+testdriver.js+%2Flink.*rel%3D.*match%2F&type=code
[6]: https://chromium.googlesource.com/chromium/src/+/main/third_party/blink/web_tests/wpt_internal/README.md
[7]: https://chromium.googlesource.com/chromium/src/+/main/third_party/blink/web_tests/resources/testdriver-vendor.js

## Details

### Usage

Using testdriver.js in non-testharness tests will be almost identical to the
testharness case.
Simply add `<script src=/resources/testdriver(-vendor).js>` before other
scripts:

```html
<html class=reftest-wait>
<link rel=match href=/common/blank.html>
<script src=/resources/testdriver.js></script>
<script src=/resources/testdriver-vendor.js></script>
<script>
  test_driver.bless("complete test",
    () => document.documentElement.classList.remove("reftest-wait"));
</script>
```

There are no other changes to (print-)reftest or crashtest markup or [test
completion behavior].

[test completion behavior]: https://web-platform-tests.org/writing-tests/reftests.html#controlling-when-comparison-occurs

### Manifest Format Changes

Similar to testharness manifest items, items for the other types may set a
boolean `testharness` field in its "extras":

```diff json
 {
   "items": {
     "reftest": {
       "testdriver.html": [
         "d890d8926273af84acf01372bd2e4fbd74d47003",
         [
           null,
           [["/infrastructure/reftest/green.html", "=="]],
           {
+            "testdriver": true
           }
         ]
       ]
     },
     "crashtest": {
       "testdriver.html": [
         "ecd7430281983f39b8269a899877ad44d760c309",
         [
           null,
           {
+            "testdriver": true
           }
         ]
       ]

     }
   }
 }
```

To avoid bloating the manifest, omitting `testdriver` implies a `false` default.

This field will let manifest consumers tell if a test requires testdriver
without needing to re-parse test files themselves.

### Lint Tool

The [`TESTDRIVER-IN-UNSUPPORTED-TYPE` rule][8] will become obsolete and can be
removed.

[8]: https://github.com/web-platform-tests/wpt/blob/290c27a84c2e630458687e757555dfe88b0cefc0/tools/lint/rules.py#L282-L284

## Risks

* Browser vendors that use custom test runners or non-WebDriver protocols may
  not be able to run the new tests without additional porting effort.
  It seems unlikely that significant proportions of (print-)reftests and
  crashtests will require testdriver in the future, so simply skipping those
  tests should not appreciably decrease coverage from the status quo.
* Opening up testdriver to the other test types may encourage authors to write
  unnecessarily JavaScript-heavy tests.
  Test reviewers should exercise their judgement in these cases.
