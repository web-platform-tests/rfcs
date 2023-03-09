# RFC 134: Bump minimum supported Python to 3.7

## Summary

Increase the minimum supported Python version to 3.7.

## Details

Currently we support Python 3.6+. However 3.6 was EOL in December
2021, and the Python ecosystem has generally moved away from offering
3.6 support. Practically this means that we have been unable to update
many dependencies which depend on 3.7+.

Previously the blocker for updates was the downstream integration with
Gecko's CI which still used Python 3.6. This has recently been updated
to use [3.7](https://bugzilla.mozilla.org/show_bug.cgi?id=1734402),
which unblocks updating to that version.

Practically this requires several manual changes:

 * Change the CI jobs running under Python 3.7 to instead use 3.7.
 * Update any outdated vendored dependencies that are being held back
   due to lack of 3.6 support.
 * Retrigger CI runs for dependabot PRs. Hopefully this allow many
   dependencies to be updated, although there may be some for which
   the latest versions require 3.8+. In these cases we should manually
   try to update to the latest 3.7 compatible release.


## Risks

3.7 itself is near the end of its support period, so there is some
risk that we end up in a similar situation again soon, with libraries
requiring 3.8+. However given the requirement to support vendors, and
the fact that Gecko depends on 3.7, bumping the minimum version
further is not possible at this time.

Some unknown external users may depend on 3.6 support. This is
hopefully reasonably unlikely as they would both need to be using wpt
in a large system where Python updates are challenging, and also not
be participating in the RFC process. However if this does happen we
will need to work with the external user to determine a viable path
forward.
