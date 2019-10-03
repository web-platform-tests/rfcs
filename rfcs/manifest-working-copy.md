# RFC 19: Always use the working copy for the manifest

## Summary

Instead of defaulting to building a manifest using what's in the git HEAD,
unconditionally use what's in the working copy.

## Details

`./wpt manifest` supports two modes, the default (based on the `git` tree at
HEAD) and `--work` (which uses the working copy). We supported the former mode
mostly as a trivial performance optimization, though I believe it is now
slower.

Many people find the (default) `git` behaviour confusing (adding a new test
won't be runnable till it's been committed, for example), and it's not clear it
gives much in the way of benefits.

We should drop support for the `git` behaviour and always use the working copy
behaviour.

We should, however, amend the working copy behaviour to, if possible, call `git
ls-tree` (or similar) to get object IDs without having to read & hash them
ourselves.

## Risks

This could make manifest generation much slower for people who have large
numbers of extra files in their working copy.

We will generate manifest items for items ignored in non-root ignore files
(https://github.com/web-platform-tests/wpt/issues/7206), which e.g. if the CSS
build system has been run could be tens of thousands of extra test files.
