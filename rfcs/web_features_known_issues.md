## Summary

This RFC proposes the creation of a new repository within the web-platform-tests organization to manage information about known issues affecting web platform tests. The repository will provide a transparent and collaborative space for documenting these issues. The data in this repository aims to be machine readable source of truth for sites or tools that make use of the web-features grouping of WPT such as webstatus.dev.

## Details

* **Repository Name:** web-platform-tests-feature-issues
* **GitHub Organization:** web-platform-tests
* **Structure:**
    * Feature-specific YAML files (e.g., `grid.yaml`) listing reasons for hiding scores.
    * An optional `fast_approvals.yaml` file for temporary, expedited approvals based on specific criteria.
* **Process:**
    1. Submit a Pull Request (PR) to add/modify feature files.
    2. Community review and discussion.
    3. Approval and merging of the PR.
* **Governance:**
    * Open to contributions from all browser vendors and the wider community.
    * Clear guidelines for approvals and time limits will be established.
* **Enumerations:**
    * To standardize reasons for hiding scores, the following enumerations will be used in feature YAML files:
        * `incomplete`: Less than 50% of the feature is tested by any reasonable coverage metric.
        * `unrelated_feature`: Issues with an unrelated feature affect more than 10% of the tests.
        * `infra`: WPT infrastructure issues affect more than 10% of the tests.
        * `other`: Other valid reason (must include a detailed description).

### Fast Approvals

The `fast_approvals.yaml` file allows for temporary approvals based on specific criteria. This is intended for scenarios where swift action is necessary, such as:

* Critical WPT issues impacting scores.
* Urgent data accuracy concerns.

Fast approvals will have a limited duration to ensure they are reviewed and addressed promptly.

#### Fast Approvals - Abuse Prevention Mechanisms
To prevent abuse and ensure transparency, the following mechanisms will be implemented:
1. Limited Duration: Fast approvals will have a strictly enforced expiration time (e.g., 7 days).
2. Protected Branch: A designated protected branch will be established within the repository (e.g., known-issues-live). This branch will serve as the source of truth for the current and past state of feature issues.
3. Automated Updates: An automated process will regularly (e.g., daily) check for expired fast approvals and update the protected branch accordingly. Only authorized maintainers or a designated bot will have permission to directly modify this branch.
4. Transparency and Logging: All fast approval requests, their duration, and expiration status will be logged and made visible to the community. This ensures transparency and allows for scrutiny if needed.
5. Rate Limiting: To prevent excessive use of fast approvals, a rate limit will be imposed on the number of times a specific feature can be fast-tracked within a given time frame (e.g., a maximum of 3 fast approvals per feature within a 30-day period). This will encourage thorough review and discourage the use of fast approvals as a workaround for the standard process.
By combining temporary approvals with a protected branch, automated updates, and rate limiting, we can ensure that expedited actions are taken only when necessary and that they are reviewed regularly to maintain the accuracy and reliability of the data.


## Risks

* **Process Overhead:** The review and approval process could introduce delays.
* **Misuse:** The system could be misused to intentionally hide unfavorable test results.

## Mitigation

* **Clear Guidelines:** Establish clear governance rules to ensure a smooth and fair process.
* **Time Limits:**  Impose time limits on both regular and fast-track approvals to prevent abuse and ensure timely review.
* **Regular Review:** Conduct periodic reviews of hidden scores to maintain data integrity.

## Rationale for Inter-Browser Vendor Collaboration

Hosting the repository within the web-platform-tests organization emphasizes collaboration, neutrality, and shared ownership of web platform data. This avoids the perception that any single vendor controls the flagging of issues and encourages broader participation in ensuring the accuracy of sites and tools relying on this information. 