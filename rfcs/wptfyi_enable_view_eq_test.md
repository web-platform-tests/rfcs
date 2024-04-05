## RFC 190: Enable view=test mode in wpt.fyi

## Summary
This RFC proposes enabling a new "test" view mode. This mode introduces a new method for counting total and failing tests, focusing on a strict pass/fail metric based on subtest counts. It includes frontend modifications to facilitate this change. This is to support this [feature request](https://github.com/web-platform-tests/wpt.fyi/issues/3740).

## Details

### Key Changes
Currently, the feature is behind a feature flag that is set to false by default in production.
* **Frontend Changes:**
   - Rename "Default View" to "Subtest View" to match the URL parameter `view=subtest` and better explain what it is. (It will still remain the default view.)
   - When looking at a run for Interop features, add a third button named "Test View". This would be in addition to existing "Interop View" and "Subtest View" buttons.
   - When looking at a run for everything else, add two buttons. "Subtest View" and "Test View"
These changes will be behind a feature flag for view=test. The default view will not change.

### Background about "test" View Mode Behavior
* Test pass is only registered when the number of passed subtests equals the total number of subtests.
* Percentages are not displayed, only raw counts at the directory level.
* Individual test level display:
    - Shows "PASS" only when subtest passes equal total subtests.
    - Shows "FAIL" when `status == OKAY` but subtest passes do not equal total subtests.
    - Existing `STATUS_ABBREVIATIONS` mappings used for other statuses.
    - "FAIL" as a fallback for any unmapped statuses.

### How To Test The Change

The feature has been enabled in staging by default so users can try it out.

- Go to any page on https://staging.wpt.fyi that shows test counts
- Add the view=test query parameter to the URL

Examples:
- https://staging.wpt.fyi/results/?label=experimental&label=master&aligned&view=test
- https://staging.wpt.fyi/results/compat?label=experimental&label=master&aligned&view=test
- https://staging.wpt.fyi/results/webstorage?label=experimental&label=master&aligned&view=test


### Risks

1. This new approach has different strengths and weaknesses to the long-standing default subtest view on wpt.fyi.
    - Mitigation: The default will remain unchanged to support current workflows and existing links.
