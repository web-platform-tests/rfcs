# RFC: File and directory picker automation in WPT

## Summary

Add `testdriver.js` and `wptrunner` support for automating file and directory picker dialogs (`<input type="file">`, `window.showOpenFilePicker()`, `window.showSaveFilePicker()`, and `window.showDirectoryPicker()`) and provisioning temporary test files and directories across both remote/sandboxed environments and desktop OS environments.


## Details

Currently, tests under `file-system-access/` that invoke file or directory pickers must be marked as manual tests because JavaScript running in automated test environments cannot interact with platform-native OS dialogs. This prevents automated cross-browser CI coverage for picker APIs and downstream filesystem functionality (including the 26 manual local filesystem tests under `file-system-access/local_*.https.html`).

Following discussions at TPAC 2026 regarding remote test grids (e.g. BrowserStack, Sauce Labs), mobile sandboxes (e.g. iOS), and OS-level edge cases, this RFC proposes an ahead-of-time (AOT) dialog automation API paired with a **two-layer fixture provisioning architecture**.

*(Note: While `testdriver.js` methods that map directly onto WebDriver BiDi commands are exempt from the RFC process per [RFC 127](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/rfc_exemption_testdriver_method.md) and [RFC 185](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/testdriver_bidi.md), this RFC documents the full end-to-end design to provide context for the `wptrunner`-managed privileged fixture layer.)*

### **1. File Picker Automation API (`testdriver.js` / WebDriver BiDi)**

* `test_driver.set_file_picker_behavior(options)`: Pre-configures the behavior of the next file or directory picker dialog. Maps onto WebDriver BiDi `browser.setFileDialogBehavior` ([w3c/webdriver-bidi#1177](https://github.com/w3c/webdriver-bidi/issues/1177)).
  * `options.action`: `"accept"` or `"cancel"` (or `null` to reset to the default behavior, which dismisses unconfigured file dialogs).
  * `options.paths`: An array of paths (returned by fixture provisioning below) to be selected when `action` is `"accept"`, and disallowed when `action` is `"cancel"`.
  * **Acceptance (`{ action: "accept", paths: [...] }`)**: Pre-configures the paths to return. Passing an empty array (`[]`), non-array, or omitting paths throws an upfront `TypeError`, as confirming zero files is not a valid OS dialog state. Providing multiple paths to a single-select picker (e.g. `showOpenFilePicker({ multiple: false })` or `showSaveFilePicker()`) cancels the dialog, causing the picker call to reject with an `AbortError`.
  * **Cancellation (`{ action: "cancel" }`)**: Simulates user cancellation or dismissing the dialog, causing the next picker call to reject with an `AbortError`.

### **2. Two-Layer Temporary Directory & Fixture Provisioning**

To support both sandboxed/remote environments (where the test runner does not share a filesystem with the browser or lacks out-of-band channels like `adb`) and OS-level edge cases (which an in-browser remote end cannot create due to sandbox and privilege limits), fixture provisioning is split into two layers:

#### **Layer 1: Standard Fixtures in WebDriver BiDi (`browser.createTemporaryDirectory`)**
* **What it does**: Exposed via `test_driver.create_temporary_directory(options)` (mapping to WebDriver BiDi `browser.createTemporaryDirectory`), this command allocates an isolated temporary directory inside the remote end's own writable space and populates files declaratively (e.g. specifying relative filename, base64-encoded content, or a zero-filled byte size).
* **Cross-platform & remote-friendly**: Works across all platforms—including iOS (inside the app container sandbox) and remote cloud grids (BrowserStack, Sauce Labs)—without requiring host filesystem sharing or out-of-band protocols. The returned path works seamlessly with both `browser.setFileDialogBehavior` and `input.setFiles`.
* **Safe & minimal**: Does not support setting executable bits (`+x`), `chmod`, symlinks, or path traversal (`..`), eliminating remote code execution (RCE) and security concerns in the standard WebDriver BiDi protocol.

#### **Layer 2: Privileged OS Fixtures in `wptrunner` (Desktop-Only)**
* **What it does**: Prepares privileged OS-level fixtures on the host that an in-browser BiDi remote end cannot create due to browser sandbox and OS privilege restrictions:
  * **Permission denials**: `chmod 000` files/directories to test `EACCES` and `NotAllowedError`.
  * **Blocked system paths**: Testing File System Access §8.1 sensitive roots (e.g. `/etc`, `C:\Windows`, `~/.ssh`).
  * **Symlinks**: Creating symbolic links (which require Administrator privileges on Windows).
  * **Executable & concurrency tests**: `chmod +x` file warnings, and external processes locking files during writes.
  * **Crash-safe cleanup**: `wptrunner` survives browser crashes and restores permissions (`chmod 0755`) upon test completion or runner exit to safely delete locked fixtures without leaking disk space.
* **Test declaration (`<meta>`)**: Tests requiring Layer 2 fixtures declare them via `<meta>` tags so that remote grids and mobile runners can skip them upfront, while co-located desktop CI environments provision and run them.

### **Open Design Question: Directory Layout Across Layers**

* **Option 1: Dedicated temporary directories per layer (Preferred)**
  * Layer 1 (managed by the browser/BiDi remote end) and Layer 2 (managed by `wptrunner` on the host) each allocate their own dedicated temporary directory. This keeps lifecycle and cleanup cleanly separated—especially since Layer 2 requires host-side permission restoration (`chmod 0755`) before deletion. Both layers simply yield paths that pass into `set_file_picker_behavior({ action: "accept", paths: [path] })`, keeping the dialog automation API orthogonal and simple.
* **Option 2: Single shared temporary directory**
  * `test_driver.create_temporary_directory()` coordinates with `wptrunner` to populate any pre-declared `<meta>` fixtures inside the same temporary directory upon creation (note that `<meta>` fixtures cannot be created before `test_driver` requests directory creation under this model).


## Protocol & Runner Integration

* **WebDriver BiDi**: Standard dialog interception and standard temporary directory creation map directly onto WebDriver BiDi (`browser.setFileDialogBehavior` and `browser.createTemporaryDirectory`, tracked in [w3c/webdriver-bidi#1177](https://github.com/w3c/webdriver-bidi/issues/1177)).
* **`wptrunner`**: Implements the BiDi protocol bindings for Layer 1 and manages host-side privileged fixture setup, `<meta>` parsing, and crash-safe permission cleanup for Layer 2 on desktop platforms.
* **In-browser harnesses**: Legacy or in-browser test runners (such as Chromium's `content_shell`) can delegate `testdriver-vendor.js` calls to internal mock dialog and temporary directory APIs.


Links:

* [WebDriver BiDi Issue 1177 (`browser.setFileDialogBehavior`)](https://github.com/w3c/webdriver-bidi/issues/1177)
* [Prototype Implementation in Chromium](https://chromium-review.googlesource.com/c/chromium/src/+/8482432) (to be updated)


## Risks

* **Protocol Alignment:** Requires standardizing `browser.setFileDialogBehavior` and `browser.createTemporaryDirectory` in WebDriver BiDi (tracked in [w3c/webdriver-bidi#1177](https://github.com/w3c/webdriver-bidi/issues/1177)).
* **Platform Coverage for Layer 2:** Tests relying on Layer 2 privileged OS fixtures (`<meta>`) will only run on co-located desktop runners where `wptrunner` shares the filesystem with the browser, though the vast majority of tests (including the 26 `local_*.https.html` tests) only need Layer 1 and will run across all platforms and remote grids.
