# RFC: File and directory picker automation in WPT

## Summary

Add `testdriver.js` support for automating file and directory picker dialogs (`<input type="file">`, `window.showOpenFilePicker()`, `window.showSaveFilePicker()`, and `window.showDirectoryPicker()`) and provisioning temporary test files and directories via WebDriver BiDi.


## Details

Currently, tests under `file-system-access/` that invoke file or directory pickers must be marked as manual tests because JavaScript running in automated test environments cannot interact with platform-native OS dialogs. This prevents automated cross-browser CI coverage for picker APIs and downstream filesystem functionality (including the 26 manual local filesystem tests under `file-system-access/local_*.https.html`).

Following discussions at TPAC 2026 in the WPT and WebDriver BiDi working groups regarding remote test grids (e.g. BrowserStack, Sauce Labs) and mobile sandboxes (e.g. iOS), both **file dialog interception** and **temporary directory/file provisioning** will be standardized directly in **WebDriver BiDi** as two orthogonal commands.

*(Note: Because `testdriver.js` methods that map directly onto WebDriver BiDi commands are exempt from the WPT RFC process per [RFC 127](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/rfc_exemption_testdriver_method.md) and [RFC 185](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/testdriver_bidi.md), this document records the end-to-end architecture across the two BiDi commands.)*

### **1. File Picker Automation API (`testdriver.js` / WebDriver BiDi `browser.setFileDialogBehavior`)**

* `test_driver.bidi.browser.set_file_dialog_behavior(params)`: Pre-configures the behavior of file and directory picker dialogs. Maps directly onto WebDriver BiDi `browser.setFileDialogBehavior` ([w3c/webdriver-bidi#1177](https://github.com/w3c/webdriver-bidi/issues/1177), [w3c/webdriver-bidi#1193](https://github.com/w3c/webdriver-bidi/pull/1193)).
  * `params.fileDialogBehavior`: `{ type: "accept", paths: [...] }`, `{ type: "cancel" }`, or `null` (to reset to the default behavior).
  * **Acceptance (`{ type: "accept", paths: [...] }`)**: Pre-configures the paths (returned by temporary directory/file provisioning below) to select when a file or directory dialog opens. Providing a non-empty array of paths is required. Providing multiple paths to a single-select picker (e.g. `showOpenFilePicker({ multiple: false })` or `showSaveFilePicker()`), or paths that do not exist or do not match the requested dialog type, cancels the dialog (causing the picker call to reject with an `AbortError` or fire a `cancel` event on `<input type="file">`).
  * **Cancellation (`{ type: "cancel" }`)**: Simulates user cancellation or dismissing the dialog, causing the picker call to reject with an `AbortError` (or fire a `cancel` event on `<input type="file">`).

### **2. Temporary Directory & File Provisioning (`testdriver.js` / WebDriver BiDi `browser.createTemporaryDirectory` & `browser.removeTemporaryDirectory`)**

Rather than provisioning files on the host in `wptrunner`, temporary directory and test file creation and cleanup are handled by WebDriver BiDi commands (`browser.createTemporaryDirectory` and `browser.removeTemporaryDirectory`, specified in [w3c/webdriver-bidi#1202](https://github.com/w3c/webdriver-bidi/pull/1202)):

* **Declarative remote-end provisioning**: Exposed to tests via `test_driver.bidi.browser.create_temporary_directory(params)`, this command allocates an isolated temporary directory inside the remote end's writable space and populates test files, subdirectories, and symbolic links declaratively (specifying relative path, `network.BytesValue` content or byte size, symbolic link target, and permissions as defined by the BiDi spec).
* **Orthogonal composition**: Returns the created directory path on the remote end, which tests pass directly into `test_driver.bidi.browser.set_file_dialog_behavior({ fileDialogBehavior: { type: "accept", paths: [path] } })` (or `input.setFiles`).
* **Cross-platform & remote-friendly**: Because provisioning happens entirely on the BiDi remote end, tests work across all platforms—including iOS (inside the app container sandbox) and remote cloud grids (BrowserStack, Sauce Labs)—without requiring shared host filesystems or out-of-band channels like `adb`.
* **Per-test and session cleanup**: Tests or `wptrunner` can remove temporary directories after each test via `test_driver.bidi.browser.remove_temporary_directory({ path })`, and the remote end also cleans up any remaining temporary directories automatically when the session ends.


## Protocol & Runner Integration

* **WebDriver BiDi**: Dialog interception (`browser.setFileDialogBehavior`, [w3c/webdriver-bidi#1193](https://github.com/w3c/webdriver-bidi/pull/1193)) and temporary directory creation/removal (`browser.createTemporaryDirectory` and `browser.removeTemporaryDirectory`, [w3c/webdriver-bidi#1202](https://github.com/w3c/webdriver-bidi/pull/1202)) are defined in WebDriver BiDi and exposed through `testdriver.js` (`test_driver.bidi.browser.*`).
* **`wptrunner`**: Forwards the `testdriver.js` actions directly to the corresponding WebDriver BiDi commands without requiring custom host-side filesystem management.
* **In-browser harnesses**: Legacy or in-browser test runners (such as Chromium's `content_shell`) can delegate `testdriver-vendor.js` calls to internal mock dialog and temporary directory APIs.


Links:

* [WebDriver BiDi Issue 1177 (`browser.setFileDialogBehavior`)](https://github.com/w3c/webdriver-bidi/issues/1177)
* [WebDriver BiDi PR 1193 (`browser.setFileDialogBehavior` spec PR)](https://github.com/w3c/webdriver-bidi/pull/1193)
* [WebDriver BiDi PR 1202 (`browser.createTemporaryDirectory` & `browser.removeTemporaryDirectory` spec PR)](https://github.com/w3c/webdriver-bidi/pull/1202)
* [WHATWG HTML PR 13197 (`<input type="file">` hook integration)](https://github.com/whatwg/html/pull/13197)
* [WICG File System Access PR 477 (`showOpenFilePicker`, `showSaveFilePicker`, `showDirectoryPicker` hook integration)](https://github.com/WICG/file-system-access/pull/477)
* [Companion WPT PR 64840](https://github.com/web-platform-tests/wpt/pull/64840)
* [Prototype Implementation in Chromium](https://chromium-review.googlesource.com/c/chromium/src/+/8482432) (to be updated)


## Risks

* **Protocol Alignment:** Requires standardizing `browser.setFileDialogBehavior` ([w3c/webdriver-bidi#1193](https://github.com/w3c/webdriver-bidi/pull/1193)) and the companion temporary directory/file creation command in WebDriver BiDi.
