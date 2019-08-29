# Remove support for stub-* files in the manifest

## Summary
Any files named stub-* are currently treated as "stub" in the manifest. Remove support for this.

## Details
Files named stub-* exist only in service-workers/, added in [Initial SW stub addition](https://github.com/web-platform-tests/wpt/commit/5ac8b780247e02235b3d8d344428886b20e5b8b0) and since then not modified in any meaninful way.

The important effect in the manifest code is that stub-* aren't treated as tests, effectively ignoring them.

The stub-* files will be removed first, followed eventually by the manifest changes.

## Risks

A MANIFEST.json file now lists the stub-* files under `manifest.items.stub`, where `manifest` is the result of parsing MANIFEST.json. The combination of an existing manifest and the new manifest code has the follow risks.

### stub-* files treated as tests

The files will be removed first, and that change allowed to propagate to all downstream users of WPT, before the manifest code change is made.

### Failure to do an incremental update

It will be confirmed that any existing `manifest.items.stub` object is dropped when doing an incremental update. Keeping it would likely be harmless, but an incremental update should always produce the same result as a full update.
