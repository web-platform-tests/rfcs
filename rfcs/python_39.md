# RFC 200: Bump minimum supported Python to 3.9

## Summary

Raise the minimum supported Python version from 3.8 to 3.9.

## Details

The project currently supports Python 3.8 and newer. Python 3.8 reached end
of life in October 2024, and the broader Python ecosystem has largely dropped
support for it. As a result, we have been unable to update several dependencies
that now require Python 3.9 or newer.

Historically, continued Python 3.8 support was required due to downstream CI
integrations with vendor infrastructure. These systems have since
been updated and no longer depend on Python 3.8, removing the primary blocker
for increasing our minimum supported version.

Implementing this change will require several concrete steps:

- Update CI jobs currently running on Python 3.8 to use Python 3.9.
- Refresh vendored dependencies that were previously constrained by
  Python 3.8 compatibility.
- Re-run CI for Dependabot pull requests to allow dependency updates to proceed.
  Some dependencies may now require Python 3.10 or newer; in those cases, we
  should manually update to the latest release that remains compatible with
  Python 3.9.

## Risks

Python 3.9 itself reached end of life in October 2025, which means we
will encounter similar pressure to raise the minimum version again as
dependencies may already require Python 3.10 or newer. Any further
increase in the minimum supported Python version should be handled
through a separate RFC, particularly to account for vendor requirements.

There is also a possibility that some external users rely on Python 3.8
support, especially those using Ubuntu 20.04 LTS with extended security
maintenance through 2030. While this scenario is considered unlikely,
because such users would need to depend heavily on wpt in an environment
where Python upgrades are difficult and also not participate in the RFC
process, we will work with affected users in such cases to identify a
viable path forward.
