# RFC 117: Remove dated tags for epoch branches

## Summary

Stop creating periodic tags for the `epochs/*` branches, such as
`epochs/daily/2022-06-30_02H` created for `epochs/daily`. Also remove all such
tags created so far.

## Details

[wpt#26122](https://github.com/web-platform-tests/wpt/pull/26122) introduced
these tags in order to help identify which commit had triggered CI runs in the
past. However, the `./wpt rev-list` command already supports listing those
commits. This will be documented.

## Risks

The `./wpt rev-list` command depends on the `merge_pr_*` tags, but the CI job
that creates them is not 100% reliable, see
[wpt#23426](https://github.com/web-platform-tests/wpt/issues/23426). If the tags
are missing but later backfilled, what `./wpt rev-list` lists can change, so
that the commits listed aren't the ones that were pushed to `epochs/*` branches.
In 2020 ~1-3% of tags were missing, and it continues to happen at least a few
times a year. No mitigiation is suggested since backfilling tags is only done
manually and rarely.

Since these tags have been created since October 2020 there is a risk that
someone has grown a dependency on them in their workflow. This RFC is mainly to
make the change visible to uncover such uses.
