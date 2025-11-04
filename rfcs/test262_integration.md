# RFC 229: Integrating Test262

## Summary

This RFC proposes integrating the Test262 (ECMAScript) test suite into WPT. Due to its large size (>50,000 files), tests will be fetched on-demand rather than vendored into the repository. Execution will be opt-in via a new `--test262` flag. This provides a unified way to run this conformance suite while protecting the WPT repository from bloat and performance degradation.

## Motivation

*   **Browser-based Testing:** Run Test262 in real browsers (stable and experimental), not just JS shells. This provides a more complete conformance signal for how ECMAScript behaves in the full web platform environment.
*   **Unified Infrastructure:** Use the existing WPT infrastructure (`wpt`, CI) and `wpt.fyi` for test execution and results analysis.
*   **Operational Efficiency:** Remove the need for vendors to maintain separate infrastructure for running Test262 and correlating its results with WPT.

## Detailed Design

### 1. Source Management

The WPT repository will contain a reference to the official `tc39/test262` repository (e.g., via a `.gitmodules` entry). This reference specifies the expected local path for the Test262 source and a pinned commit SHA.

*   The pinned commit SHA ties each WPT commit to a precise version of Test262, ensuring reproducible test runs.
*   An automated process (e.g., a weekly GitHub Action) will create pull requests to update this pinned SHA, providing a regular cadence for ingesting upstream updates.

See Section 3 for details on how the Test262 source is acquired.

### 2. Test Integration

WPT's manifest generation and serving logic will be extended to support the on-demand nature of Test262 tests.

*   **Manifest Generation:** The `wpt manifest` command will recognize the Test262 source directory and traverse its `test` directory to discover tests.
*   **Metadata Parsing:** A new, Test262-specific parser will read the YAML frontmatter from each `.js` file to extract metadata (e.g., ES features, negative test expectations, execution flags).
*   **Harness and Server:** Specialized Test262 harness files and `wptserve` handlers will use the parsed metadata to construct the correct execution environment and generate distinct URLs for different test modes.

### 3. Acquiring Test262 Source Code

The Test262 source code is acquired on-demand. `wpt` can handle this automatically for users in a Git environment, while providing a manual path for CI systems.

When the `--test262` flag is used, `wpt` first checks if the Test262 source directory exists. If missing, `wpt` determines if the main WPT repository is a Git checkout.
- **If it is a Git repository,** `wpt` will attempt to run `git submodule update --init --recursive`.
- **If it is not a Git repository,** `wpt` will fail with an error, instructing the user to acquire the source manually.

This allows CI systems and vendors with non-Git checkouts to use custom tooling, while providing a seamless experience for the common user.

### 4. Execution Control

Running Test262 is **opt-in**.

*   **Default Behavior:** By default, `wpt run` will not look for the Test262 directory and will not include its tests.
*   **Opt-in Flag:** A new `--test262` flag will be added to `wpt run`.
*   **Flag Behavior:** When present, `wpt` ensures the Test262 source code is available (fetching it if possible, per Section 3) and includes the Test262 tests **in addition to** the standard WPT tests.
*   **CI Integration:** CI systems can enable Test262 runs by adding this flag, after ensuring the source has been acquired.

## Alternatives Considered

### Using `--test-types=test262`

This was rejected because Test262 is fundamentally different from internal WPT test types (e.g., `testharness`, `reftest`). It is an external suite with its own metadata format and harness. A dedicated `--test262` flag provides a clearer separation of concerns and a cleaner implementation.

## Implementation Considerations & Risks

*   **Vendor Integration:** The responsibility for acquiring the Test262 source is left to the user/CI system. This allows vendors with non-Git source control to implement their own sync mechanisms.
*   **CI Submodule Behavior:** Some CI systems may check out submodules by default, causing an unintentional, resource-intensive checkout of the large Test262 repository. The impact and configuration options for common CI systems require investigation.
*   **CI Performance:** A full Test262 run is lengthy. It is anticipated that full runs will be scheduled on a nightly or weekly basis, not on every pull request.
*   **Initial Source Checkout:** The first-time fetch of the Test262 repository will cause a one-time delay.
*   **Source Code Visibility on wpt.fyi:** Linking to test source on `wpt.fyi` is not straightforward. A solution would require `wpt.fyi` to have special-case logic for Test262, including access to a manifest mapping test names to file paths and knowledge of the pinned submodule SHA for a given run.

## Proof of Concept

A prototype implementation exists in the following commit:
[https://github.com/o-/wpt/commit/baa8415800161122e7d9d1170513f5aad82a4504](https://github.com/o-/wpt/commit/baa8415800161122e7d9d1170513f5aad82a4504)

*Caveat: The YAML parsing logic in this prototype will be moved to a more Test262-specific location.*
