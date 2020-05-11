# RFC 41: Print Testing

## Summary

Add a test type for testing paged media.

## Background

We've [long
aspired](https://github.com/web-platform-tests/wpt/issues/7760) to
have some form of visual testing for paged media. While fragments can
be tested to some degree with features like multicol (with
`break-before`/`break-after` support, etc.), there are some features
specific to print that we want to test (`@page` especially).

There has been a lot of prior work here:

 * Gecko supports
   [`print`](https://searchfox.org/mozilla-central/rev/ad6234148892bc29bf08d736604ac71925138040/layout/tools/reftest/README.txt#304-348)
   reftests and
   [`reftest-paged`](https://searchfox.org/mozilla-central/rev/ad6234148892bc29bf08d736604ac71925138040/layout/tools/reftest/README.txt#650-679),
   which do two different things. The former runs the actual print
   codepath (to PDF) but only does a very partial comparison (page
   count and (non-rasterirzed) text) of the PDFs. The latter runs a
   slightly different code-path that results in a sequence of PNGs
   (one per page) which then get compared. (Note that until 2017 only
   the latter existed, previously as
   [`reftest-print`](https://bugzilla.mozilla.org/show_bug.cgi?id=1382327).)
   (The [bug](https://bugzilla.mozilla.org/show_bug.cgi?id=374050)
   that introduced `reftest-paged` gives no rational as to why it
   doesn't use PDF or similar, rather adding a custom code-path; the
   [bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1299848) that
   added `print` starts with saying it should compare with the
   rasterized output but then it gets resolved with very different
   behaviour.)

* WebKit's history has two halves:
  * [#15558](https://bugs.webkit.org/show_bug.cgi?id=15558) added
    support for PDF reftests on 2008-07-09. This was then skipped for
    the "mac" platform (the one platform it wasn't already skipped on)
    in [r35135](https://trac.webkit.org/changeset/35135/webkit) two
    days later due to
    [#20011](https://bugs.webkit.org/show_bug.cgi?id=20011) (PDFs
    differ between machines). That bug proposed visually diffing the
    PDFs instead, but ultimately…
  * [#37203](https://bugs.webkit.org/show_bug.cgi?id=37203) added
    support to render in print mode to a PNG (where multiple pages are
    separated by a horizontal rule). Part of the reason for moving
    away from PDFs was, "as it would require some amount of efforts to
    make it portable among [WebKit] ports".

  The latter support survives in both WebKit and Blink, and is triggered by a
  call to `testRunner.setPrinting()`.

We previously blocked
[#5381](https://github.com/web-platform-tests/wpt/issues/5381) on
stopping classifying all the existing print tests as manual, given
moving all the existing print tests to `-manual` and then having to
later move them again seems less than ideal, and there was a hope this
would get solved Real Soon Now® several years ago. We should at least
agree what we're doing with print tests, even if we don't initially
have any implementation (and every single runner yields "unsupported
test type").

## Details

This proposal adds a new test type, called `print-reftest`.

This new test type is similar to the existing `reftest` type; it uses
`<link rel=match>` and `<link rel=mismatch>`, combining in the same
way as for normal reftests. It also supports `reftest-wait`. Instead
of comparing screenshots of the viewport, a print-reftest produces a
paginated view of the document, and compares a bitmap rendering of
each page in the test to the corresponding page in the ref. In order
to pass an equal comparison test must have all pages the same whereas
a not-equal comparison test must have at least one page different.

This tests are identified by a `print` type-flag in the filename
(i.e., "foo-print.html").

The paper sizes for the pagniated layout are 5 inches by 3 inches with
0.5 inch margins on all sides. Initially this is fixed, but this could
later be relaxed by adding a new meta property to specify the required
dimensions.

In some cases it may not be desirable to compare all pages of the
output. To support this use case a new meta value is used; `<meta
name="page-ranges">`. The content value of this element follows the
form:
```
content = (RefPath ":")? Range ("," Range)*
Range = Int | Int "-" Int
Int = [0-9]+
RefPath = /* Relative path to a reference of the current test */
```
Multiple `<meta name=page-ranges>` elements may be specified in a
single test, each applying to a particular reference. A range with no
reference specified applies to the test itself. Ranges specified in
reference files are an error that are ignored. The `page-ranges`
specifier for a particular document is interpreted as for the [parse a
page range](https://www.w3.org/TR/webdriver/#dfn-parse-a-page-range)
algorithm in the WebDriver spec.

### Implementation in wptrunner

The cross-browser implementation of this feature relies on two things:
* The [print endpoint in the webdriver
  standard](https://www.w3.org/TR/webdriver/#print)
* pdf.js providing a cross-browser mechanism to render a PDF without
  depending on external tools, many of which are not cross-platform or
  have inappropriate licensing.

The WebDriver-based implementation first renders the document to a
PDF, and then computes the reftest result by rasterising the test and
ref PDFs at 96 dpi.

This stack is good enough to prove that the feature can be implemented
in any browser that has the relevant standards support, and will be
enough to run the feature in CI and get results on wpt.fyi. Vendors
may wish to use a custom implementation in their own infrastructure
for reasons of performance. Indeed if the browser is able to rasterise
directly to the paginated form that will avoid the need to create an
intermediate PDF.

## Risks

* Browsers' rendering to PDF is non-deterministic and therefore
  identical pages won't be guaranteed to yield a PDF that is
  graphically the same on each load. This can be addressed with
  appropriate fuzzy specifiers, or by using browser-specific
  alternatives to PDF generation.

* We don't catch non-visual differences in the output, such as
  (accessible) text content in the document. This feature is supported
  by gecko's print reftests, so we may be missing some testing
  needs. This could be fixed by additional meta elements to either
  mandate that the text has to match or to provide some text strings
  that ought to be in the output file. Such a feature would prevent
  optimisations that avoid producing a PDF entirely, but may be
  important for ensuring that browsers don't incorrectly rasterize
  text in their print output (although such a thing is strictly hard
  to test in a cross browser fashion since there's no standard for the
  internals of the produced PDF).

* The default implementation rendering the PDF using pdf.js inside the
  browser could result in a bug affecting both the test and the
  rasterization. But these use very different codepaths and layout
  techniques, so the chance of a bug affecting both seems small
  compared to a bug affecting both the test and the ref.
