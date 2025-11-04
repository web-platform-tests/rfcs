# RFC XXX: Integrating Test262

## Summary

This RFC proposes the integration of the official ECMAScript test suite (Test262) into the web-platform-tests (WPT) project. Given the suite's substantial size (over 50,000 files), this proposal avoids vendoring the files directly into the WPT repository. Instead, Test262 will be integrated via a mechanism that ensures tests are fetched on-demand, as detailed in Section 3, "Acquiring Test262 Source Code".

Execution of these tests will be strictly opt-in, controlled by a new `wpt` command-line flag. This approach provides access to a critical conformance suite, enables unified test execution and analysis, and protects the WPT repository from excessive bloat and performance degradation.

## Motivation

*   **Comprehensive Conformance:** Test262 is the authoritative test suite for ECMAScript (JavaScript). Its inclusion provides a complete and standards-aligned signal for JS engine conformance, which is a core part of the web platform.
*   **Unified Infrastructure:** Integrating Test262 allows browser vendors to run this essential suite using the same mature infrastructure (`wptrunner`, CI) and analyze results on the same platform (`wpt.fyi`) as all other web platform features.
*   **Operational Efficiency:** This removes the need for vendors to maintain separate infrastructure for running Test262 and attempting to correlate its results with WPT. It streamlines the engineering workflow for testing and triaging JS engine behavior.

## Detailed Design

### 1. Source Management

To ensure reproducibility without vendoring the entire suite, the WPT repository will contain a **reference** to the official `tc39/test262` repository. This reference will specify the expected local path for the Test262 source code and a pinned commit SHA.

*   The pinned commit SHA guarantees that any given WPT commit is tied to a precise version of Test262, ensuring repeatable test runs.
*   An automated process, such as a weekly GitHub Action, will create pull requests to update this pinned SHA. This provides a regular, managed cadence for ingesting upstream Test262 updates.

For details on how to acquire the Test262 source code based on this reference, please refer to Section 3, "Acquiring Test262 Source Code."

### 2. Test Integration

The existing WPT manifest generation and serving logic will be extended to support the on-demand nature of the Test262 tests.

*   **Manifest Generation:** The `wpt manifest` command will recognize the Test262 source directory. When the source is present, the command will traverse its `test` directory to discover tests.
*   **Metadata Parsing:** A new, Test262-specific parser will be used to read the YAML frontmatter from each `.js` file. This parser will extract critical metadata, such as ES features, negative test expectations, and execution flags (e.g., `strict`, `module`, `raw`).
*   **Harness and Server:** The specialized Test262 harness files and `wptserve` handlers will use the parsed metadata to construct the correct execution environment for each test, generating distinct URLs for different test modes.

### 3. Acquiring Test262 Source Code

It is the responsibility of the user or CI system to ensure the Test262 source code is present at the location specified in the `.gitmodules` file. `wptrunner` will not perform the sync itself. This provides flexibility for different development and CI environments.

There are two primary methods for acquiring the source:

*   **Method 1: Git Submodule (Recommended)**
    For users in a standard Git environment, the tests can be synced by running `git submodule update --init`. This will fetch the precise Test262 revision pinned in the main WPT repository.

*   **Method 2: Manual/Custom Sync**
    For CI systems or vendors using non-Git source control, the Test262 repository can be acquired through other means, such as a direct `git clone` or a custom mirroring process. The system must ensure that the repository is checked out to the specific commit SHA defined in the `.gitmodules` file and placed at the correct local path.

### 4. Execution Control

To prevent performance degradation for users not focused on JS engine testing, running Test262 will be **opt-in**.

*   **Default Behavior:** By default, `wpt run` will **not** look for the Test262 directory and will **not** include its tests in the run.
*   **Opt-in Flag:** A new command-line flag, `--test262`, will be added to `wpt run`.
*   **Flag Behavior:** When the `--test262` flag is present, `wpt` will verify that the Test262 source directory exists. If the directory is missing, `wpt` will raise an error with instructions on how to sync the code (as described in the "Acquiring Test262 Source Code" section). If present, it will include the tests in the run.
*   **CI Integration:** CI systems can explicitly enable Test262 runs by adding this flag to their configuration, after ensuring the source has been acquired.

## Implementation Considerations & Risks

*   **Vendor Integration:** The responsibility for acquiring the Test262 source code is deliberately left to the user. This allows vendors with non-Git source control systems to implement their own mechanisms for syncing the required Test262 revision.
*   **`WEB_FEATURES.yml` Mapping:** The current mechanism for mapping tests to features relies on `WEB_FEATURES.yml` files being co-located with the tests they describe. Since the Test262 source will not be in the main WPT repository, a new mechanism will need to be devised to associate these tests with web features for analysis in tools like `wpt.fyi`.
*   **CI Performance:** A full Test262 run is lengthy. The impact on CI resources and runtime must be monitored. It is anticipated that full runs will be scheduled on a nightly or weekly basis rather than on every pull request.
*   **Initial Source Checkout:** The first time a user runs with `--test262`, they must ensure the Test262 source is checked out. The one-time delay of fetching the repository should be clearly documented.
*   **Source Code Visibility on wpt.fyi:** A key trade-off is that `wpt.fyi` cannot easily link to the source code of Test262 tests. The necessity of this feature and potential solutions (e.g., linking to a harness page with a note) will need to be discussed.

## Proof of Concept

A prototype implementation of the required harness, server, and manifest changes exists in the following commit.

**Link:** [https://github.com/o-/wpt/commit/baa8415800161122e7d9d1170513f5aad82a4504](https://github.com/o-/wpt/commit/baa8415800161122e7d9d1170513f5aad82a4504)

*Caveat: As part of the final implementation, the YAML parsing logic (`parseTestRecord.py`) will be moved to a more Test262-specific location (e.g., `tools/test262/`) to clarify that it is not a generic WPT manifest parser.*
