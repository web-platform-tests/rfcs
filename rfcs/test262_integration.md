# RFC 229: Integrating Test262

## Summary

This RFC integrates the Test262 (ECMAScript) test suite into WPT by vendoring it directly under `tc39/test262`. Execution is opt-in via `--test-types=test262`.

## Motivation

*   **Browser-based Testing:** Run Test262 in real browsers (stable and experimental), not just JS shells. This provides a more complete conformance signal for how ECMAScript behaves in the full web platform environment.
*   **Unified Infrastructure:** Use the existing WPT infrastructure (`wpt`, CI) and `wpt.fyi` for test execution and results analysis.
*   **Operational Efficiency:** Remove the need for vendors to maintain separate infrastructure for running Test262 and correlating its results with WPT.
*   **Interop:** Allow ECMAScript features to be proposed and selected for inclusion in cross-browser Interop efforts.

## Detailed Design

### 1. Source Management

The Test262 test suite will be vendored directly into the WPT repository under `tc39/test262/`. This path mirrors external test suites like WebAssembly (`wasm/core/`), with `tc39/` serving as a namespace for TC39-originated tests.

*   A dedicated CI job, similar to `update-wasm-tests.yml`, will keep this vendored copy up-to-date. It will periodically:
    1.  Checkout the official `tc39/test262` repository.
    2.  Copy the `test/` and `harness/` directories into `tc39/test262`.
    3.  Open a PR against `main`, noting the upstream commit SHA in the body for traceability.
*   The update PR will include a smoketest to ensure Test262 integration is not broken. This small, dedicated test suite in `infrastructure/test262/` will verify key integration points (e.g., YAML frontmatter parsing for `negative` tests/`flags`, `assert.js` harness injection). Expected results will be in corresponding `.ini` files within `infrastructure/metadata/test262/` (e.g., `infrastructure/testharness/lone-surrogates.html` metadata in `infrastructure/metadata/infrastructure/testharness/lone-surrogates.html.ini`). This smoketest is the *only* blocking Test262-related check in the PR; full test runs occur in scheduled nightly runs, populating wpt.fyi.
*   This approach ensures WPT consumers have no novel requirements beyond selecting the test type.

### 2. Test Integration

WPT's manifest generation will be extended to support Test262.

*   **Manifest Generation:** `wpt manifest` will, by default, recognize `tc39/test262`, traverse its `test` directory, and add discovered tests to `MANIFEST.json`.
*   **Metadata Parsing:** A new Test262-specific parser will read YAML frontmatter from `.js` files to extract metadata (e.g., ES features, negative expectations, execution flags). Unlike `wasm/core` tests, which are pre-processed, Test262 metadata will be used by `wptserve` handlers at runtime.
*   **Harness and Server:** Specialized Test262 harness files and `wptserve` handlers will use parsed metadata to construct the execution environment. Test modes will use distinct URL conventions based on file extensions (e.g., `path/to/test.js` served as `path/to/test.test262.html` or `path/to/test.test262-module.html`).

### 3. Execution Control

Running Test262 is **opt-in**.

*   **Default Behavior:** `wpt run` will not include Test262 tests by default, even if they are present in the manifest.
*   **Opt-in Flag:** The existing `--test-types` flag selects Test262 tests.
*   **Flag Behavior:** When `--test-types=test262` is present, `wpt` includes Test262 tests from `tc39/test262`.
*   **CI Integration:** CI systems enable Test262 runs by adding `--test-types=test262` to their `wpt run` command.

## Alternatives Considered

### Storing Test262 Metadata in the Manifest

Storing parsed YAML metadata from Test262 files directly within `MANIFEST.json` was considered. This is the standard WPT architecture for other dynamic test types (e.g., Workers, Window tests) to optimize runtime performance.

The current dynamic approach requires two disk I/O operations per test: one for the `wptserve` handler to parse YAML frontmatter, and a second for the browser to fetch the `.js` content. Storing metadata in `MANIFEST.json` would eliminate the first disk I/O and YAML parsing, effectively halving disk I/O operations per test during runtime.

However, this optimization is deferred for initial implementation simplicity. Metadata will be parsed dynamically by `wptserve` handlers at runtime.

### Pre-generating HTML Wrappers

Pre-generating static `.js.html` files for each Test262 test (as done for some other vendored suites like WebAssembly) was considered but rejected due to:

*   **Complexity of Multiple Contexts:** Test262 tests execute in various contexts (e.g., module, strict). Pre-generating wrappers for all combinations would lead to a combinatorial explosion of static files, adding potentially far more than 50,000 new files and creating a significant management burden.

Dynamic HTML wrapper generation at serve-time, driven by Test262's metadata, offers a more flexible and efficient solution without duplicating repository files.

### Using a dedicated `--test262` flag

This was considered to provide a clear separation from WPT's internal test types. However, using the existing `--test-types` flag is more consistent with how other test types are selected and is the preferred approach.

## Implementation Considerations & Risks

*   **Manifest Generation Performance:** Including the 50,000+ Test262 tests in the manifest by default will increase the time required to run `wpt manifest`. This performance cost is accepted to ensure a simpler user experience, where a single, default manifest contains all tests.
*   **Repository Size:** Vendoring Test262 will significantly increase WPT repository size (~50,000 files), a manageable increase.
*   **CI Performance:** Full Test262 runs are lengthy. They will be scheduled during the existing nighly builds, not on every pull request.
*   **Upstream Changes:** Updating vendored Test262 tests requires care to ensure changes to the test harness or metadata format are handled correctly. The CI job's update PRs will include a smoketest for basic integration issues.
*   **Unit Tests for New Code:** New Python code for Test262 integration (e.g., manifest parsing, serving logic) will have dedicated unit tests, separate from integration smoketests.

## Proof of Concept

A prototype Test262 integration exists in:
[https://github.com/web-platform-tests/wpt/pull/55997](https://github.com/web-platform-tests/wpt/pull/55997)
