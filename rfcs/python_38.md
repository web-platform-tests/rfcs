# RFC 189: Bump minimum supported Python to 3.8

## Summary

Increase the minimum supported Python version to 3.8.

## Details

Currently we support Python 3.7+. However 3.7 was EOL in June 2023,
and the Python ecosystem has generally moved away from offering
3.7 support. Practically this means that we have been unable to update
many dependencies which depend on 3.8+.

Previously the blocker for updates was the downstream integration with
Gecko's CI which still used Python 3.7. This has recently been updated
to use [3.8](https://bugzilla.mozilla.org/show_bug.cgi?id=1843209),
which unblocks updating to that version.

Practically this requires several manual changes:

* Change the CI jobs running under Python 3.7 to instead use 3.8.
* Update any outdated vendored dependencies that are being held back
   due to lack of 3.7 support.
* Retrigger CI runs for dependabot PRs. Hopefully this allow many
   dependencies to be updated, although there may be some for which
   the latest versions require 3.9+. In these cases we should manually
   try to update to the latest 3.8 compatible release.

## Risks

3.8 itself is near the end of its support period (October 2024), so there
is some risk that we end up in a similar situation again soon, with libraries
requiring 3.9+. However given the requirement to support vendors, and
the fact that Gecko depends on 3.8, bumping the minimum version
further is not possible at this time.

Some unknown external users may depend on 3.7 support. This is
hopefully reasonably unlikely as they would both need to be using wpt
in a large system where Python updates are challenging, and also not
be participating in the RFC process. However if this does happen we
will need to work with the external user to determine a viable path
forward.
