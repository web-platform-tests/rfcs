# RFC 229: Integrating Test262

## Summary

This RFC proposes integrating the Test262 (ECMAScript) test suite into WPT by vendoring it directly into the repository under `tc39/test262`. Execution will be opt-in via the existing `--test-types=test262` flag. This provides a unified and simple way for consumers to run this conformance suite.

## Motivation

*   **Browser-based Testing:** Run Test262 in real browsers (stable and experimental), not just JS shells. This provides a more complete conformance signal for how ECMAScript behaves in the full web platform environment.
*   **Unified Infrastructure:** Use the existing WPT infrastructure (`wpt`, CI) and `wpt.fyi` for test execution and results analysis.
*   **Operational Efficiency:** Remove the need for vendors to maintain separate infrastructure for running Test262 and correlating its results with WPT.
*   **Interop:** Allow ECMAScript features to be proposed and selected for inclusion in cross-browser Interop efforts.

## Detailed Design

### 1. Source Management

The Test262 test suite will be vendored directly into the WPT repository under the `tc39/test262/` directory. This path structure follows the precedent set by other external test suites like the WebAssembly tests, which are located in `wasm/core/`. The top-level `tc39/` directory serves as a namespace for tests originating from that standards body.

*   A dedicated CI job will be responsible for keeping this vendored copy up-to-date, similar to the `update-wasm-tests.yml` workflow.
*   This CI job will periodically:
    1.  Checkout the official `tc39/test262` repository into a temporary location.
    2.  Copy the relevant Test262 source directories (`test/` and `harness/`) into the `tc39/test262` directory within the WPT repository. WPT's manifest generation and serving logic will then interpret Test262's metadata and dynamically configure the test environment at runtime.
    3.  Create a new branch, commit the updated Test262 files, and open a pull request against the `main` branch. The body of the pull request will note the full commit SHA from the `tc39/test262` repository that is being vendored, ensuring traceability.
*   The update PR will include a smoketest to ensure that the Test262 integration has not been broken by the upstream changes. This smoketest will consist of a small, dedicated suite of tests located in a new `infrastructure/test262/` directory. These tests will live permanently in the WPT repository and will be designed to verify the key integration points between `wptrunner` and the vendored Test262 files, such as correctly parsing Test262's YAML frontmatter (e.g., for `negative` tests or `flags`) and ensuring Test262's harness files (e.g., `assert.js`) are properly injected and functional. The expected results for these smoketests will be defined in corresponding `.ini` files within the `infrastructure/metadata/test262/` directory, following the established WPT metadata pattern where each test file can have its own metadata file (e.g., the test `infrastructure/testharness/lone-surrogates.html` has its metadata defined in `infrastructure/metadata/infrastructure/testharness/lone-surrogates.html.ini`). Crucially, this smoketest is the only Test262-related suite that runs as a blocking check in the update pull request. The newly vendored tests themselves are executed against browsers later, as part of the regularly scheduled full test runs (e.g., nightly), which populate the results on wpt.fyi.
*   This approach ensures that WPT consumers have no novel requirements to run these tests, beyond selecting the new test type.

### 2. Test Integration

WPT's manifest generation will be extended to support Test262.

*   **Manifest Generation:** The `wpt manifest` command will recognize the `tc39/test262` directory and traverse its `test` directory to discover tests. All discovered tests will be added to the main `MANIFEST.json`.
*   **Metadata Parsing:** A new, Test262-specific parser will read the YAML frontmatter from each `.js` file to extract metadata. This is a key difference from other vendored suites like the `wasm/core` tests, which are pre-processed into self-contained HTML files. Test262's reliance on YAML frontmatter necessitates a metadata-driven approach. This metadata (e.g., ES features, negative test expectations, execution flags) will be used by the `wptserve` handlers at runtime to dynamically configure the test environment.
*   **Harness and Server:** Specialized Test262 harness files and `wptserve` handlers will use the parsed metadata to construct the correct execution environment. Different test modes will be served via distinct URL conventions based on file extensions (e.g., a test at `path/to/test.js` might be served as `path/to/test.test262.html` for a normal test or `path/to/test.test262-module.html` for a module test).

### 3. Execution Control

Running Test262 is **opt-in**.

*   **Default Behavior:** By default, `wpt run` will not include Test262 tests.
*   **Opt-in Flag:** The existing `--test-types` flag will be used to select Test262 tests.
*   **Flag Behavior:** When `--test-types=test262` is present, `wpt` will include the Test262 tests from the `tc39/test262` directory.
*   **CI Integration:** CI systems can enable Test262 runs by simply adding `--test-types=test262` to their `wpt run` command.

## Alternatives Considered

### Storing Test262 Metadata in the Manifest

Storing the parsed YAML metadata from Test262 files directly within the `MANIFEST.json` entry for each test was considered. This approach would offer significant performance benefits by avoiding the need to re-read and re-parse the YAML frontmatter from each `.js` file at runtime when a test is served. However, for the initial implementation, this optimization is being deferred to prioritize a simpler manifest generation process. The metadata will instead be parsed dynamically by `wptserve` handlers at runtime.

### Pre-generating HTML Wrappers

An alternative to dynamic HTML wrapper generation at serve-time, as done for some other vendored test suites (e.g., WebAssembly), would be to pre-generate static `.js.html` files for each Test262 test during the vendoring process. This approach was considered but rejected for the following reasons:

*   **Repository Bloat:** Test262 contains over 50,000 `.js` test files. Pre-generating a corresponding HTML wrapper for each would double the number of files, significantly increasing repository size and potentially impacting filesystem performance.
*   **Complexity of Multiple Contexts:** Test262 tests can be executed in various contexts (e.g., as a raw script, as a module, in strict mode, non-strict mode). A single `.js` file might need to be served with different HTML wrappers depending on the desired execution mode. Pre-generating all possible combinations would lead to a combinatorial explosion of static files and introduce significant complexity in managing and updating them.

Dynamic HTML wrapper generation at serve-time, driven by Test262's metadata, offers a more flexible and efficient solution by constructing the appropriate test environment on-the-fly without duplicating files in the repository.

### Using a dedicated `--test262` flag

This was considered to provide a clear separation from WPT's internal test types. However, using the existing `--test-types` flag is more consistent with how other test types are selected and is the preferred approach.

## Implementation Considerations & Risks

*   **Repository Size:** Vendoring Test262 will significantly increase the size of the WPT repository. While the number of files is large (~50,000), this is a manageable increase on the current repository size.
*   **CI Performance:** A full Test262 run is lengthy. It is anticipated that full runs will be scheduled on a nightly or weekly basis, not on every pull request.
*   **Upstream Changes:** Care must be taken when updating the vendored Test262 tests to ensure that any changes to the test harness or metadata format are handled correctly in the WPT integration. The CI job that opens update PRs should include a smoketest to catch basic integration issues.
*   **Unit Tests for New Code:** Any new Python code developed for Test262 integration (e.g., for manifest parsing or serving logic) will be covered by its own dedicated unit tests, separate from the integration-focused smoketests.

## Proof of Concept

A prototype implementation of the Test262 integration exists in the following pull request:
[https://github.com/web-platform-tests/wpt/pull/55997](https://github.com/web-platform-tests/wpt/pull/55997)
