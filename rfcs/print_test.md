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
  
* There is [ongoing
  work](https://github.com/w3c/webdriver/issues/1093) to add print
  support to WebDriver.
  
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

This proposal is to add a new test type, provisionally called `print`.

This new test type would be very similar to the existing `reftest`
type; it would similarly use `<link rel=match>` and `<link
rel=mismatch>`, combining in similar ways to how they do for
reftests. It would also similarly support `reftest-wait`.

The different with the existing `reftest` type is that instead of
taking a screenshot of the current viewport, this would render the
document to a sequence of PDF pages. The two lists of PDF pages would
first be compared for length (i.e., if there are different numbers of
pages, the test fails), and then each page in turn is rasterized and
compared with reference.

This test type would be discovered by a `print` type-flag (i.e.,
"foo-print.html").

## Risks

* Browsers' rendering to PDF is non-deterministic and therefore
  identical pages won't be guaranteed to yield a PDF that is
  graphically the same on each load.
  
* We don't catch non-visual differences in the PDFs, such as
  (accessible) text content in the document.
  
* The system we use for rasterizing the PDFs to compare each page is
  non-deterministic and therefore yields different PNGs (AIUI, if we
  use ghostscript, any such case would be considered a bug in
  ghostscript).
