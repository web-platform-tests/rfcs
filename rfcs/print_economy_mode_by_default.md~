## RFC #xyz: wptrunner to print in economy mode by default

### Summary

Change to print (or rather generate print layout for) web tests in economy mode
by default, which means that backgrounds will be skipped by default. Currently
wptrunner forces backgrounds when printing.

### Details

Most browsers print without background (colors and images) by default, as
including them would typically use a lot more ink / toner. Then browsers
typically offers a checkbox or similar to always force backgrounds, if that's
what the user prefers. This is about changing wptrunner to behave like browsers
do by default.

There's a CSS property `print-color-adjust` specified at
https://drafts.csswg.org/css-color-adjust-1/#print-color-adjust , whose initial
value is `economy`, which means to print without backgrounds. Setting it to
`exact` will include backgrounds. If the browser is set up to always print
backgrounds, this property will have no effect, since the spec says that the
user preference must be respected more strongly than the hint provided by
`print-color-adjust`. And now we're getting to the point: Without this change,
it's impossible to test this property in web platform tests.

Adding a META flag to force backgrounds in web platform tests nevertheless is
possible, but not planned at this point, as it shouldn't be necessary, since
it's outside the scope of CSS. The goal is merely to be able to test that
`print-color-adjust` is handled correctly.

Pull request for wptrunner:
https://github.com/web-platform-tests/wpt/pull/51825

### Risks

This may break assumptions in existing vendor-local tests that aren't
upstreamed, that still use wptrunner.

All tests that have been upstreamed to WPT, on the other hand, have already been
prepared for this change, by specifying `print-color-adjust:exact` where
backgrounds are tested.
