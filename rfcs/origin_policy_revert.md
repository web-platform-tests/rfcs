# RFC #113: Remove Origin Policy infrastructure

## Summary

Support for testing [Origin Policy](https://wicg.github.io/origin-policy/) was introduced in [RFC #44](origin_policy.md) + [wpt PR #21705](https://github.com/web-platform-tests/wpt/pull/21705). Revert this code, removing the 99 op* subdomains, for example op4.web-platform.test.

## Details

Origin Policy is no longer being pursued. The tests using this infrastructure were removed in [wpt PR #33875](https://github.com/web-platform-tests/wpt/pull/33875).

## Risks

There could be tests using the subdomains for something other than testing Origin Policy. To catch any such cases, all instances of "op" followed by a digit will be inspected, and full runs of wpt with the infrastructure removed will be compared to the current state.
