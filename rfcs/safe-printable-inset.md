# RFC #233: Simulate safe printable inset

## Summary

We need a way of testing the [safe-printable-inset](https://github.com/w3c/csswg-drafts/pull/13190/files) property.

## Details

Most printers have a small region along each edge of the paper edges that's not
reliably printable, usually due to the printer's paper handling mechanism.
Authors can steer clear of such unprintable areas using the
`safe-printable-inset` property, which applies in `@page` and `@page` margin
contexts.

There should be a way for print reftests to test this, by simulating unprintable
areas.

One rather straight-forward solution would be a META value that sets the width
of the unprintable area on all four sides. For instance:

`<meta name="safe-printable-inset" content="[inset-specifier]"`

where `inset-specifier` is a numeric value. The unit could be CSS pixels or
points. Using centimeters for anything here isn't a great idea, since they don't
convert nicely into CSS pixels (unlike inches). I suggest using CSS pixels.

Why just one value for all four edges? Although many printers indeed don't
necessarily have a uniform unprintable area width along each of the four paper
edges (although many do), so that just providing one value for all is an
oversimplification of reality, printers may rotate the print output at their own
discretion. The user agent may therefore not be able to make assumptions about
which edge (long or short?) will be fed first into the printer, or what
orientation the sheet of paper has. Therefore using just one value (which should
represent the largest of the four) seems reasonable.
