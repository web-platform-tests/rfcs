# RFC 122: Remove browser specific failures from wpt.fyi

## Summary

[wpt.fyi](https://wpt.fyi) currently shows a chart of "browser
specific failures"; a score of tests that are failing only in
a single browser. This RFC proposes entirely removing that graph
from wpt.fyi.

## Details

The original motivation for browser-specific-failures was to provide
browser vendors with insights into tests that might be causing interop
problems and therefore might be especially valuable to spend
engineering effort fixing. On the basis of this hypothesis, a graph
was added to wpt.fyi showing a browser-specific-failures "score" for
each browser engine, so that vendors could track their progress on
fixing these issues.

However at this point we have identified a number of issues with
"browser specific failures" as a metric, for example:

* It's hard to correlate a change in the score to a change in browser
  behaviour.

* The way the score is computed biases the score toward missing
  features rather than interop failures in already shipping features
  (since a missing feature usually causes a large number of failures).

* The metric doesn't provide any way of controlling for the user
  impact of failures; browsers can get a "bad" score from a large
  number of failures that in practice don't cause any observed
  problems for authors.

For these reasons and others, we haven't reached a critical mass of
adoption for browser specific failures as a metric to improve interop
on the web platform, and its original function has been largely
replaced by the Interop-20xx project.

Although browser specific failures isn't providing the initially hoped
for value, having it on the wpt.fyi does create some work as it
encourages people to try to understand the current scores or changes
in the score.

Removing the graph entirely seems like the simplest way to indicate
that we no longer consider this useful as a metric.

## Alternatives

* Keep the graph but move it to a less prominent page.

  Although this would make the graph less obvious, it would still
  imply some endorsement of browser specific failures as useful at the
  level of a metric. Since Interop scores are a metric that browser
  vendors have explicitly committed to, and which solve many of the
  problems with browser specific failures, it's better to clearly
  commit to one public metric.

## Risks

* Browser engineers might be using browser specific failures as a way
  to identify good tests to fix.

  This doesn't depend on having a metric / graph of browser-specific
  failures, and could be better solved by making it easier to see an
  actual list of browser specific failures in a given feature. This is
  already possible on wpt.fyi, but is quite complex to write as a
  query. Other frontends on wth wpt.fyi data like
  https://jgraham.github.io/wptdash/ provide a engineer-focused view
  of this data.
