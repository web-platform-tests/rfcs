# RFC: File and directory picker automation in WPT

## Summary

Add `testdriver.js` support for automating file and directory picker dialogs triggered by the [File System Access API](https://wicg.github.io/file-system-access/) (`window.showOpenFilePicker()`, `window.showSaveFilePicker()`, and `window.showDirectoryPicker()`).

&nbsp;

## Details

Currently, tests under `file-system-access/` that invoke file or directory pickers must be marked as manual tests because JavaScript running in automated test environments cannot interact with platform-native OS dialogs. This prevents automated cross-browser CI coverage for picker APIs and downstream filesystem functionality (including the 26 manual local filesystem tests under `file-system-access/local_*.https.html`).

This RFC proposes adding an ahead-of-time (AOT) pre-configuration API to `testdriver.js`, paired with runner-side path resolution and temporary directory provisioning:

### **1\. File Picker Automation API (`testdriver.js`)**

* `test_driver.set_file_picker_behavior(options, context = null)`: Pre-configures the behavior of the next file or directory picker (one-shot).  
  * `options.action`: `"accept"`<!-- gdoc_comment_begin: c1 --> or `"cancel"`.  
  * `options.paths`: An array <!-- gdoc_comment_end: c1 -->of relative paths to be selected when `action` is `"accept"`, and disallowed when `action` is `"cancel"`.  
  * **Acceptance (`{ action: "accept", paths: [...] }`)**: Pre-configures the paths to return. Passing an empty array (`[]`), non-array, or omitting paths throws an upfront `TypeError`, as confirming zero files is not a valid OS dialog state. Providing multiple paths to a single-select picker (e.g. `showOpenFilePicker({ multiple: false })` or `showSaveFilePicker()`) causes the test harness to fail at dialog invocation.  
  * **Cancellation (`{ action: "cancel" }`)**: Simulates user cancellation or dismissing the dialog, causing the next picker call to reject with an `AbortError`.

### **2\. Path Resolution & Temporary Directory Provisioning (`wptrunner`)**

Because `testdriver.js` is intended for user/browser interaction rather than OS filesystem management, and to avoid exposing host OS absolute paths to JavaScript tests, tests pass **relative paths** to `set_file_picker_behavior` while `wptrunner` resolves them to absolute host paths:

* Following the established WebExtensions pattern in `wptrunner` (using `testdriver.js?feature=...` or path matching such as `file-system-access/local_*`), `wptrunner` automatically provisions an isolated writable directory in the system temporary folder <!-- gdoc_comment_begin: c2 -->(via Python's `tempfile` module)<!-- gdoc_comment_end: c2 --> before the test starts and resolves relative paths (e.g. `paths: ["."]`) under that temporary directory.  
* For read-only tests that do not request a writable temporary directory, relative paths are resolved against the test's source directory (matching `WebExtensionsProtocolPart._resolve_path`).  
* Deterministic cleanup of temporary directories is handled by `wptrunner` upon test completion or runner exit, preventing source tree corruption, parallel-run flakiness, and disk leaks even if browser tests crash.

&nbsp;

## Protocol & Runner Integration

* Cross-browser execution maps cleanly onto the proposed WebDriver BiDi **`browser.setFileDialogBehavior`** command ([w3c/webdriver-bidi\#1177](https://github.com/w3c/webdriver-bidi/issues/1177)), where `{ action: "accept", paths }` (after `wptrunner` resolves relative paths to absolute host paths) maps to `type: "allowed"` with `files`, and `{ action: "cancel" }` maps to `type: "denied"`.  
* In-browser test harnesses can also delegate directly via `testdriver-vendor.js` to internal mock dialog APIs.

&nbsp;

Links:

* [WebDriver BiDi Issue 1177 (`browser.setFileDialogBehavior`)](https://github.com/w3c/webdriver-bidi/issues/1177)  
* [Prototype Implementation in Chromium](https://chromium-review.googlesource.com/c/chromium/src/+/8482432) (to be updated)

&nbsp;

## Risks

* **API Surface:** Minimal impact on `testdriver.js`, adding only a single extensible `set_file_picker_behavior` method while keeping temporary directory provisioning and path resolution internal to `wptrunner`.  
* **Protocol Alignment:** Requires standardizing the ahead-of-time file dialog behavior command in WebDriver BiDi (tracked in [w3c/webdriver-bidi\#1177](https://github.com/w3c/webdriver-bidi/issues/1177)).  
* **Remote / Mobile Environments:** Runner-side temporary directory provisioning and path resolution are scoped to co-located desktop CI where the runner and browser share the filesystem; remote device bridging (e.g. Android) can be added as a future transport extension.

&nbsp;

---
gdoc_comments:
  c1:
    status: ACTIVE
    token: AX5pRSnzSr5X9dpDHhe0zwhyAqki9hnSigBF0DVRBk0Ln12CXbtdJn39aDDal4E_x2JbLl4xdpkPlbuZPMV42BIQJ1OMSGV1ILwblDTpLz46N7ynmYYJ68qE_FZOkM5N-fkWsSp5dIPLDQDwrSyh_x5leUcPk3a-1zlT8MTVX_OHdh6iTSQfV1eL4H6lRmhlUS1ikl-fQWzf2t99U6utGfaqWr2qixBG-CmzjTLwZcSw0ur8ZEARfPuLq9TsgCwWXli_y2Pp8JIBorfwRLwQy7DB4rrHM0uRQACkAPSc9dJCbN3QXUWAqRPmboKIl015H-d-A40sQMS0Y94z_0KN0tJDi4fp6gkq-XchB669WpYTPZ_RSjAJngsRlFWfA1wMfyJg3V6CgJ0Wrt_G6rV-OQcc-_D2ATMWysixvLZB4pgbDNXIHAW8w_NJrp6XdIqabmsPWalUOcmIa1-wMhe8TAfxHWfNZwzkkj52niBJmvyoQrWD2g2kki-fm0DDsBqrZWelc1-ZfCrtZjqC8P8CHNWwU5LZhHn16S_5kW52x00JLTp4Y5Te6zTtjgDwZ6XkKPMQDJadwjdD3EyKjdncASbWWLtnmXA_Bvmi4khQqgaz9l8FXp0Nny5QNJBYBpu7So85B3J8kS9QEA4Z-yQt7QVpBvcb3oRoLNqVxe9376DzHGnbg6qzo6OfpnPfWWvCFoH9cVB6HLxolpNjnBfHnkfaR1ZyLrtLZH2IDdSvKKXEuPQ33s4wr2o4FgjMiwynddBp71u425DtAwRpWub_XBnz3CoxUaEJ5K9Ue8lHCJDYDFhB11KiXqTSy3GzWXDFZ-pYPKrv7UdIZpl0rNkYfn_xnUEryj7IDTXrqRZGZFWjPJI2xH3EvsfdqQjsloN2SVz2TFs0qE4h_F92yyCRGuPBEru8u49hqPL3bR6pq0oQI4Y
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.640Z'
      body: 'Currently, neither `action` nor `paths` is always required, yet passing
        nothing ({}) is invalid. While I understand we want to prevent the API from
        being misused, this feels a bit confusing to me.&lt;br&gt;&lt;br&gt;How about
        making `action` always required (&amp;quot;accept&amp;quot; | &amp;quot;cancel&amp;quot;),
        and making `paths` required only for &amp;quot;accept&amp;quot; (and disallowed
        for &amp;quot;cancel&amp;quot;)? That way there is only one canonical form
        for each case:&lt;br&gt;- Accept: { action: &amp;quot;accept&amp;quot;, paths:
        [...] }&lt;br&gt;- Cancel: { action: &amp;quot;cancel&amp;quot; }'
      quote: " or `&quot;cancel&quot;`.  \n  * `options.paths`: An array "
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.641Z'
      body: Agreed.&lt;br&gt;Making action always required  ensures a single canonical
        form for each case (which also matches WebDriver BiDi&amp;#39;s discriminated
        union schema).
  c2:
    status: RESOLVED
    token: AX5pRSn7JT5WZd3v4RIHbSoVMJVG1w9UmLgWX8arjWrvScKjbAejIvVHa_DZqYiWLLmO2eXcuvkt0L9iISJLLxocKRcxizxKBdNwBUQaedHt7gYFgKLr_j7brfsonYaBETeTtD8B7BTRjtKyoS2YGs2AT7XKmqRTOqnKP8OtmSy8LHO7qjiO-ajyGaf36dYPm-jPkPBhfXfEBrL_ysr4lAfAPME3ozh7_64KxNyrc4SzaT-x5ALb-cD5USBac9cd4nkr9e8l-GwySqYCwjFskUatqSo_xyRmt4k7MKPnuQ293PGLGNOFmFuhhHQc3v7-LNZDTmIOwYcH21tJ6Y9IUgC3DD9meTIesb9LO-Whkr235dfojxpKMp4n8YybSpdyn2L9C0WGxHKIfFLd3w-sLsngRJIck7EKb70Tm7kgTID_Ipp7e18Qe4TIf_9OX-fLdVoFcKMS8OZkJyXqmMEYlgLBtGcgT81JQWmQjKc-BFYvqwD_XALACwLe97IKfVgYVpCtFt2geYqNMW3X74hDT7H0LkHMXyiP9xJHvN900CFkTwP7z5s_rRdh1sZDFw5ZrSkAkMY8tYiuRmUsNgGXc-kLI5wtKkuySR0V1gKqb2tcYr9ksVd-YlOkrf4phIctAPhRffby9BKwAJQfWXin2bAvj6umfuf1HMaLvwpLwUGUVx-ie9iF_AnRvMC9B-vsrQosOfLU4NJGZN-NfX8wmQO6bIgjNEhrSja4xAfLduvjCFt_J3vojbCkNOM3cb3w9kKxKZhOwFiZsrsnQKPwQft7enJHYUk7M58MDoWPEHicLlNAGkvXnR3DqifuJjBGReQjnUCSYaMuYIMBDcrW9MPb_87WxVJUJCZ5Gx01B0Jqtca4IBU
    head:
      author: Anna Sato
      time: '2026-10-16T06:00:36.796Z'
      body: '@<a href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>
        Just to double-check: I remember we talked yesterday about possibly including
        the test file path in the temp dir name, but as long as this  `tempfile` guarantees
        a separate unique directory per test, this should be fine, correct?'
      quote: (via Python&#39;s tempfile module)
    replies:
    - author: Eriko Kurimoto
      time: '2026-10-16T08:38:20.773Z'
      body: I think so. If the dir is per-test, they should be identical and independent.
    - author: Anna Sato
      time: '2026-10-16T08:46:08.993Z'
      action: RESOLVE
  c3:
    status: RESOLVED
    token: AX5pRSmB2cjzHHKBqojY9nOfmFV86Xlp-jVK-i7HaTIT3abwo4ljn8WTQ0Po1P3v8H6BzzlQHqICAmPXDdyyMWh7DrmZMeY7K-43Qin8HPP8jrfECJ2h3WyImI4iQOUh9tn4CySpkTOayXMuRZ8KUjGdZZlyxyrafw-UN-7Mf9IP_78fMimAIdfGyZudRbnYPtMYvR8xK7J91YW_1AHKk1p4s1lxQt5os5HMlHhjbcLCNvDuWtYbyJ4Fyattiq3RHPWQ0jfxp4w2j5a5IJgfbt-1hNDgPdW6j8cwnQAtIuRO5jhKWyJMq10CySdYW6bbb0OCjBl8T8jv3y4EEPgCar6go2qgZ2PL6ZiDREHEt8d2pU1U2sWBBq0hZXZfrnrjjaQ7GcVdewfmurwmYGdAnmquP5oHSSPZ3FzX8Pp-NoVyAbhDdEgrCdWCXqezAS35vchQ5Qg9qauVHYiHrNGRY8OMA3OtzfpmOmq8wH3CCvB-KBS9tHeOezQ8xyu6PW4wj0CPiYLPCi6R80YoQj5ZVGQUQQeqZd6INrw_tpnt7a44_gbgT2NjYxm5cvUazfYf0qUnJUmacARrKuswJCIrC_8cq3kNn31iSN5aZW_wejYeJHyJJzFjzT27PCy6Fm66LpPWKvSkb71wJx-rLo5R6p_KTTG3EeNJz7yZJS0UG1VFCI-4CcFOrW77mqgzir7dx-cW9BivwKwYDq_ct6Ms0bdP4uze00XZuKhZw1kA9DrafmFXPiAEkbAQGL0mpDzOTgeLDxWI3A3IlkGVJ2Ddv01ch70I0EY5riEFyJszF7iPSRtNQ1XRATslwbm2ls0sFq1waAmzEyEN_OOAb2QaUJQNy0tMh_Xb7UnExPOroabg6M_LXuzKFAHKSRK-IQeMYMysm-yrvgt0bU4Giezncze4apnrNSKEneR29iVIDAu5BxlXxxWEouOGXCEBt2juHNA3_c9-1P4xUUKpwM-DjVdH-Mu8185WzEwAk9PO9yIWA-r03NuYAADTR4UwhkxyjluqF-x8LrygCsy0FkjCrhingjp8wvTFq1Lg6DK8bRjSsjeo3deR083qrERzOIm7RePJmTvTLiQoR_Xmn_8FZm7d3oWux68VqKGl6E4hhpM2FYt1T0ME5aFnps4rQoYsN9mCEipIU9zlXc0CKVY-vh8UMjqvNPXEY7aLhvHozfM-n6bIa11pgYZvnyXn0hC_llJUYVn0KV2NBP4sfxNJP0P1GcZHcQRM4K6IuqBkl6DGf8UbtAcz7T0GLArZAWauln7yDqwEZ3myE4j8DWDtGEHwtEH5K5M6rxFVT8NpZNX8WdStNIz4-lCd88PaLg3caZPq7jywpBdS5YkBtmrJ2Zv5hsTT88A6F3hDAuPUBL3pz-STOrmra3bmRBt1MkpNnsT-tVI_5f9pv33UQzZLPefWjREKT4Ti23J2VFMA9UdjTyeEMkATD9gHSEATQtFvPM3xGL7T19SgTaufEavindiGqi5-_oi4Hz7WZhfpzaVDFXvdXSrUKae3YCaw3Ddhui4ARrFZlQYvzSF66yG44JeyGtlZN06dC421ym27nab6c5nLKXkzg1GwTTZ7M0yOXB7Z7zy1Dsh6PMDNzj7s1zKx40A8OVFfcRQN7xlmtFmJICf04QotpB6nfVKB6Bz_-bWnWrH6zXRGYWQuBViyhLcy1RBIUP9Ghh3N
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.641Z'
      body: If we have a separate API for cancellation, we don&amp;amp;#39;t want
        to accept `null` here?
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: 'The current Chromium implementation accepts null under the hood (set_cancel_file_picker()
        is implemented by passing null).&lt;br&gt;&lt;br&gt;Keeping null accepted
        in set_file_picker_result(null) gives some flexibility—for example, in data-driven
        / parameterized tests where iterating over a test matrix with paths: null
        avoids an extra if/else branch.&lt;br&gt;&lt;br&gt;Do you think it&amp;#39;s
        better to strictly reject null (throw TypeError) to enforce using the separate
        cancellation API, or keep it allowed for flexibility while promoting set_cancel_file_picker()
        as the primary idiomatic way?'
    - author: Anna Sato
      time: '2026-10-16T04:51:47.643Z'
      body: Hmm I personally feel that if we allow set_file_picker_result(null), we
        probably don&amp;amp;#39;t need a separate set_cancel_file_picker() API at
        all.&amp;lt;br&amp;gt;&amp;lt;br&amp;gt;For parameterized tests, since a cancel
        test asserts `promise_rejects_dom` while a selection test inspects the returned
        handles, the test body needs an if/else or a separate promise_test anyway?
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.644Z'
      body: '@&amp;lt;a href=&amp;quot;mailto:&lt;a href=&quot;mailto:<a href="mailto:elkurin@google.com"
        target="_blank">elkurin@google.com</a>&quot; target=&quot;_blank&quot;&gt;<a
        href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;quot;
        target=&amp;quot;_blank&amp;quot;&amp;gt;&lt;a href=&quot;mailto:<a href="mailto:elkurin@google.com"
        target="_blank">elkurin@google.com</a>&quot; target=&quot;_blank&quot;&gt;<a
        href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;lt;/a&amp;gt;
        what do you think?'
    - author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.645Z'
      body: As chatted offline, I think having 2 different ways to do the same thing
        is not good, so we should decide which one to go. I don&amp;#39;t have a strong
        opinion on which to keep, so I&amp;#39;ll wait for your new rfc!
    - time: '2026-10-16T04:51:47.646Z'
      action: RESOLVE
  c4:
    status: RESOLVED
    token: AX5pRSnXVNgDXbXPORVOWjZBaaX5_2CAM2Gs668wRsYVNEwXQkyiESYubGDhQyi28A5OGg2rRRMtYx0xHDujgUPvixQSUwKprZgB0vgWDQV0CUTSI_ebOzbvoK4JWL8zyPNv_pky3QIjRywhyIvu1TxbNCEpVxKE3oYG3YRzbkykPjL-kNPuRxS4vi3usazNExjzpfoRhl6uCblpbasYASmW1mvY98q5m8GOTDEi3Dp1aOA0EWuETFez1zBvp5BszCf9GUtZ0LoIJPOEoC1Mx6BxJPtVybhJDfnsYt7oZrenYHeIIGdZkUYf1eGgbZSBVonalKIBuNSeBhvL-Lb09trouyX1_hyDP5MraeSYo63DuW-pt6j1KG9ZHovwNEAmhO-zomFZnlI_lsvei2IULyl_TEhge08mS35Rb2ifJPd2-SejxYXTkrzc7-BRUgVBUyQMbLHwDXvWYtoL3IeuBAmS8LMtORygfabtu0GCSYc1W2FbZJjB6HR0xleDqKH_c5EEi6F6Hgt0iWzCPtmzNA
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.641Z'
      body: Thsi sounds like the result applies to every later picker. How about &amp;quot;the
        next file or directory picker&amp;quot; / &amp;quot;the next picker call&amp;quot;
        instead?
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: updated
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c5:
    status: RESOLVED
    token: AX5pRSmdkfIF3koSwHJT97UyV8usChjTjWBsCg37UYWE1IQ5JO_wlozmnbA6Qmwe7Ab4pH3C2nB6AM2vjlI9RjTUo25au4iGYZybAe90alY-cyM1nQVVU_0akbQuKAUvNgA_Eog5V1CmKhNU3x9Mb83NS7MncJzuV8Y9pgaCKPBj5_fxiUWTi95DNI5Y3_wzZvVD1IC3W1sb9b8Wd4ckHsRfNqglLCGUY79z_-vUd_IMP8KTonofggAKimkvRjnaeLrQKCSIr0rTVaEMbmS7zd2srxDb8hT3eACXkBTWZwZxuxFoexxI1Yuyr5TJ3n7sjA0z2XuwZcH1cLT8QA57T_ZfLOfjOQYJfX8QnKO8y3UEIrFecglx8EVJMwJAFLCabRZlfrucIA95Di42bHPuNsfoalsjJyhPAqjx9MxFSA-iOmlEZhvOhZqP1rkNjvitURTutIynM_aWD2uE0JWYqx021WhRAIKb3nQWQvUFbu57moqe1RqVwzDcEEMYc2gk9lMlB_-zQSyJR-WFYxzVrbYOxg
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.641Z'
      body: Thsi sounds like the result applies to every later picker. How about &amp;amp;quot;the
        next file or directory picker&amp;amp;quot; / &amp;amp;quot;the next picker
        call&amp;amp;quot; instead?
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: updated
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c6:
    status: RESOLVED
    token: AX5pRSmojxgz5TnE4ZYe8a661X6RZycf-RCBEjiz0pW83tuznpHfN6OHGyN9woJWcPRy6pi3jMRJH_KA08nbqJLstDyUaDUyogNP6jX3maWv3Tt251EBwLM4d2fKAxFSfE7R-Ws7mzFAV5d_8kxzA-FmsVfoRafu-PYuEo7IHig2VibHQWX2hOys0oJZmuWQLtsEH66arWSVxZD5Con43IMSf73Kuh0icH4xqLnH74ULA2sEpoqNkw420JBoJrYrhZnKBcLZLG4uBpbZTMmXcTNCYjDiUKO0d4Czm5Xdw2r2b34YVvNKtyQRm8u3oV37nuqaqygnNHJXKgYy1-pHEqZ-ixrWcCBY8t8xncLcXUr4mqUuL53RwDO-pgo5IQ6siA5qZD8ZaHB3mWmKaRXEfCz-Wfg2kiIrKrKw7TebyzSuexe3T_ikPFubOSQ2rYKHi3_DBWkjfkicJwDIXczdgFx1an0
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.641Z'
      body: We should also mention that we need to introduce (or extend) fileDialogHandled-ish
        event
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: Done
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c7:
    status: RESOLVED
    token: AX5pRSnwfMgs_9oViZTOgAfcZPD0sFf8d1BrM6xjuxgwSC4n-lmi87GTRErBkGbmrkMxPSm3y2mk-5PunQfDzbVTywPq_5YZ4XG8vjtMVQIWTZMlmt_2bgZltdgIhFJbBs-UZq5CGAquRK13fBam-5cVXniCZvP9emmjN1YSVIjB8PvTSpXCDNaWsEFEPWW7g--fHBHIr8N2bGlpA9zWHe4DH5uRmExFfyT-FwKiHwgC0FYBf2eELLWAb3vYDy_MwoHz3LJ4eMo9btuM_X-BosDLuss7EORox6BqaivdFkQrw6xEz_d1qv9O7IC1nAMQ0qyXm8R77cKfzGgbAETDjk3ndkRjNRZMDWXH9hFlWMTCFTTDKOiuXDmiv4POY87WOImv0tSrM41fnlHOfudNSQaxWJKbnyr07wzWsMcZyTT53EHD2y7D0WfgjKsDJVUHNKgNAg4TsXCnkrTTWe4LnkAxCdEJb6ZR7oilhA7bj8rwqVLllMGZQPP-2teH8KN9R334thwZF9FY8cpEe0mD
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.641Z'
      body: Thsi sounds like the result applies to every later picker. How about &quot;the
        next file or directory picker&quot; / &quot;the next picker call&quot; instead?
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: updated
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c8:
    status: RESOLVED
    token: AX5pRSlu42GXTJAkCzDy6UhfvFroqgVmIXayrczzQgtdNMgtF3BZGr3Tv_pzY-ZjmsCj_mZ7TX7gDBx3s6MZRuprCVhivK4LDc9DqXEvU6enZx7769ZIclx9YQTlVR_Xy0HBjoOI_qZSiFvQzF1mcfGMmq2RX1LG9PsC4ySEPY12BHOlvtJm51Q_oLo6jAMcPRAfbacWBYBOTRZaTvxHzcOvQHDcPCVNkA-CO4CzmonZLo7g4doyOTvvLnDnEHW_5-FRhDW2Z4mukCfOaOW_eD1q0LmieaDCRpYee88AE5axOJThzPpqGnHzYKZAgKgWkJVNeRK8Gb4QkNFFbeHoJWJn8TFdUWxzGFYJJef5jRLPgwJQjgfJMXO-INDYuQ4kXFXtqFbNFzgiF4EHFqAAky3slLRhGA-lbsbW4CVTDHsr1UIn0aXQmyhHVwX-dVsmie_WQ5m1cSR4gBoWYbJ_PAccIdMM7msdtIOXd2m6xr3CjzGNQ8eJjZckcbpasSKRw99jf_vuKPU1IomzGufpP8Opv2nMPzg7wJ44yzFtias3yvQkfwaP075Ue5HWiQrhOcA1nfonwoxJ2TAd9RUsKM2zFZHqVK22W5q2yvdT0IxpUnHudrrRbq8NSdd3M3DJAEcu5uwMB9_cra3PI8icXAdw0EKS_EhBUWq4Zsz4QUonLotVdz9WLDPzbK91PbCfRutdsy2sbOtFk2fBsRzq4N4ubXbLLL6Z4h-ZBJqSICs9AX3cSTblp8FFqwU1npOdHg4ikArwAj2EnA256alF3Yq6mF-5zGa0gfuPEYlDF7pDl9hjCiFC8NwVYI4AJbHWjOXM5HZdZtOY6Nta8aXrgXbiqf7EWuNGJBbjKeWAGPTGUu9Z0U_1c2wQX91y0QuH0vp1vVb6fUOS-gnhpLx0iyK1XjuU8ezbZapmq0JLDz6gausTeVbOl2xMKx7Mzvyeoi4nz5use93IP7xMe1cNrGX1HEPMUdpxBAkOXwx81vS5mT7dEo8TrP4o20PWlp-h5-e0zwfvbDwCWp62VuisbQu7KwQdn8kxF92SebKrnsYmDecsv41WLewbyaV1DdhTa2fW-h5v5knxm_L9oA04MhW_lpxYYut8Stuj76wF-qK90YWPIWWP54gc12XFa0wWZraBlhARHUkZEHPc0Gw6p6P2HYZs_wtLAbr8igm17vCrTg
    head:
      author: Anna Sato
      time: '2026-10-19T04:42:40.149Z'
      body: Is this re-added?
      quote: File System Access API Specification
    replies:
    - author: Ayu Goto
      time: '2026-10-19T04:49:00.772Z'
      body: I haven&#39;t removed this line
    - author: Anna Sato
      time: '2026-10-19T04:52:17.666Z'
      body: <a href="https://www.google.com/url?q=https://docs.google.com/document/d/18Q6KdulI1PNExjTGHLb5S5Ii5qrWg4Jf/edit?disco%3Ddoco.pbVvPgRz049EXI2_9v0_6w&amp;sa=D&amp;source=docs&amp;ust=1792389137668014&amp;usg=AOvVaw0hJ6zb6HztoWv2pZ3G4eN_"
        data-rawhref="https://docs.google.com/document/d/18Q6KdulI1PNExjTGHLb5S5Ii5qrWg4Jf/edit?disco=doco.pbVvPgRz049EXI2_9v0_6w"
        target="_blank">https://docs.google.com/document/d/18Q6KdulI1PNExjTGHLb5S5Ii5qrWg4Jf/edit?disco=doco.pbVvPgRz049EXI2_9v0_6w</a><br><br>I
        thought we decided to remove this per this thread? Anyway, this is completely
        a nit, so feel free to ignore if you&#39;d like :)
    - author: Ayu Goto
      time: '2026-10-19T04:55:24.628Z'
      body: Sorry, I overlooked your previous comment. Dropped the link
    - author: Ayu Goto
      time: '2026-10-19T04:55:42.641Z'
      action: RESOLVE
  c9:
    status: RESOLVED
    token: AX5pRSlDErzvicjtMxqlWcCuoEYsMZrDRyExg--vW04PwafepM59fDLkXUPinVFJ4a4OVMWeWNbcoL-sGoVPTxiyc8VS1S6nQywrVOLd9KaEivMA1SVgMjM6frDAN3s8qhRbm_Q1GlW6IRiAB_nvKzDOB4W9LrDBzX7U6038uB001FlmlZEMkJ28QWCQB5c9TuI_ShLP376BDHksGZVmTaMPYOch69NqcC5-TvkiqBhUzI6feJ6gpddxCpN6JBdsGBVYcwX9EKGc7bEmdAYEqhmjigaBZ0Xfu9x89FmnXRhLFZlipWzw2FZga06egCh-jSDSDJBJ_UvkmCmaxVpU8piWFA7X7G-pWT5faBsIyYLy-uP08GmEm9NNwoR-n9L_kHtNnczRywAQw6W4rFwZYc1IVxzUWhGmOAjOOHi47UVNzmi5FUu5vrItDpLzqvEBz5ODmw_mrWmsVGkDImibL51-0CNOhWHBIQlmytilG8RUQqGZKC5qkiU7QJZCrWMuQ4O7HwjKqv7gFyjzJwMs0kQ6AR92-OMwK6awPdr74aeAqZtxXR3QxkmE-mWtUbj9AjDu4bLfGxV-uI0M
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.641Z'
      body: Let&amp;#39;s mention that passing [] is treated as an error.
    replies:
    - author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.642Z'
      body: Also nice to talk about the error cases that this API rejects and how:&lt;br&gt;e.g.
        multiple file inputed while the API only accepts single
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c10:
    status: RESOLVED
    token: AX5pRSn1SPSPwmtgMX2sfBUnu58t30j2c99Fn99hxUTtcLycD_kWZNlS4L3roCc_9C4ZLA732QkwUI4y23l6N_diZ5yt0YPXq1DejKu1Y-su4CVj9Vgc09eh5TflJyCcoG64IdynTah4J4OInpQyzvcBBRM43nQXVuOBO98VZ0ErJT79n3KzsxpeevUELQHidnqAarUGDS3W-WohV_26d0AEY8VEJOWnTrmX5cPZgmo0J2EHTKXHhryFwsvVwkOz-XrNVpilOGSbGvopHtnGCfNcAbaJp2zXrx6ZGoSLAg5r3zrTlninY_eY391svia1frEHt3rzydzeimgE6VDeJnXchPmeluOpUYRJSMVqOHN1DkHhEFffKpY0Ag-Fa2Kk6s2BgRnSCIlcpuiRu5Q_FLWyDPSFafsqLf-vuhDlzwAX-a_G7jzdcPV79HgqF4tngexZ1buBZ0ae5mh5E2_2dTn8X92u044MiDtObayp89pBqucSOk7pfG5LT3VTz-WnTlKzfLS02Q5zgmaTkM9zqCrG3Jn84CGE2IB5p8Oc7Kb1fHd_5cQ5DcD5laPAiXC_6LWAR9KznaFmb6T7UgA2pP-wMo9HJ74SXcivbSJXOKcDkRyoIQC84AII8vb5wQiwmAv0LgC7TRB9dw
    head:
      author: Anna Sato
      time: '2026-10-16T05:54:16.383Z'
      body: 'nit: It&#39;d be nice to link a spec change PR here once we have one,
        but for now I think we can just drop this link.'
      quote: File System Access API Specification
    replies:
    - author: Ayu Goto
      time: '2026-10-16T06:34:21.759Z'
      body: Agreed
    - author: Anna Sato
      time: '2026-10-16T07:35:42.776Z'
      action: RESOLVE
  c11:
    status: ACTIVE
    token: AX5pRSnuJkyYErb2p6Ns8EZjNIPdtmqX_FEoTMg0ah275L1M654YEoM4sX4sQ8aWRMPC4RR2zfHSZAgo-BPBu6r9u3WuxKWqPEvW7L9nw5-Hj-N5OuH_lGJ_hPzRW5py7We6WBU7p4nTdaFDLyXR8XtaZqMF9BswK870oVWk9EtVvqzCyY-aLEag8aTWw2YQnF7VwcKKpJ3icrw91CKMGe2tuZc-mI4mCcN02cG-wR4k8CjlLNGkDJzNa_2Dl9O94ioQ3n8RFtEe9UQwFa39QrzuHThuo_ZRytUBoOEYsXUwDnL_QRhH8L0_xpowQRhCpnRGwiRnA4TfttBVTMT7Bdx7nBG-qIdEMQcFCW5Q7lMWSyfNRVnRnGZxMUDwAWhnzc4edlzA3bHlzexAl97HCXDXvlAl_KvFNkFXhDRt3h69640xFinvUEoJzQTIarOvoBEW87lHc15TOsqO2bIt1TvKxCokPyohQ_Odp7JWqJUwhXCvgmjGkm9HSl7o3yK0N1UcOchV1GGUUC6uLQI4UhZ0Zuz0Y8QLwEwwj-rIyB31uwQfnYMYK2ZNiddHS-TTFppKKpG1_O2XPRuTIH6f5b3ywQbPvGujgkYkS6Z4VWnn_yizfF43_HgkiHsI3mRXlV9oHL4r2H2yycg
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.641Z'
      body: Let&amp;amp;#39;s mention that passing [] is treated as an error.
    replies:
    - author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.642Z'
      body: Also nice to talk about the error cases that this API rejects and how:&amp;lt;br&amp;gt;e.g.
        multiple file inputed while the API only accepts single
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
    - time: '2026-10-16T04:51:47.644Z'
      action: REOPEN
  c12:
    status: RESOLVED
    token: AX5pRSninod-fxuAl820ySm9AqDTk-uecavHW_1hN3ch8TkOXAONZXTK4PULL9HWzC0xoss2JDFaVhDAuMb40MEwrSdUUY_fSY9FTbZQSvmUkHMA88yTe_bmjLYue1l4_NCoRnPcMzk2Qu4x46R_eBB9-UDJ0oRBWNpQJtjooeLGhhePcgd0cc8O2hetHnZEr4W2yyqGgWh29NGdwWFukdqRai6pCnZ96TDmr7R75g7V6NKw13pYGf53P7pUUoQj0_BrOE6YjE4Hx3cJDJJMyYte0aoog4vBq53uQR6iMZx38cTGYluYZkZxcTTLuFSUOV7_A0GyVrtY5VIBD4xCwT-34WzWS1snLireaH0fvjg6WwbC83PKQW0pXfnKOIOc6dTjDV7wjk6vgOKymX-ZtXHhhl6tsob6pa125rCrSoTR0Lf7fIInZgArYij9n6r_tvUIsSeXJX69aWz9z2od-2ua0x_k0x_TmV-cnndVzmCRkGPUaD9BVZHZbYlcjyR6dTe8S9shB0_Z9L97DVB8mo_OL2OkRwBKPHMsodwWOXjr3T42QbP9qge06nbqxky3JrOk0NZ8MZcLoCLq
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.641Z'
      body: Let&amp;amp;#39;s mention that passing [] is treated as an error.
    replies:
    - author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.642Z'
      body: Also nice to talk about the error cases that this API rejects and how:&amp;lt;br&amp;gt;e.g.
        multiple file inputed while the API only accepts single
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c13:
    status: RESOLVED
    token: AX5pRSkmeu54XqDl9k42gH_MDZ6sdvpB9y3F-sVybWlXh-jiahh2A4jc4qMpyBSeww3glCQ5_QD3pSwj1WOcM8-k9wUmykQB7U9iJXMtqMy2zbtayoh4qvyBCQ9wfxwLFcbUSwLQSM5lkxDTe5JKEazpIcY-9okwYTJvytAsBS6qfvnUVezuBAfu76XpF2ooAhRSu1Vzs4ncrzGhHbkEHw6NySU9WuYAyXw1BwWVaXT2hfztj4uBMPAdvcvUXsdoivuvW1zxxPn8wRfXV_T6gp98H-W6yl03tzDMs4JtxB35Jp1TLWTPZsgy5LXTlnjxYu1nRxzDgCjRdGUFAN8yVPj6TfEGx4SmBN_oEsSL31tppgvvZI4ZKmJhYU3Bk3p3rcwgiSU0amEZnLCVWsXp2rRteU0IBQK130MMYeJBH4pfV9R-cSBjt1-2XbiLZsyFH4BsBtg26dr-6COCQs23NT9TGODdOQ9fkjsQiXzD0ydjuVkB_3pPr5ZXmdtFZ1Z--8raS82-dL0
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.641Z'
      body: Good call! +1 to separate the RFC. This is the most riskful change that
        may be rejected by other vendor.&lt;br&gt;Nice!
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: Thanks!
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c14:
    status: RESOLVED
    token: AX5pRSmj5xa2YCk3_R429dfn_jgfaOYlFe6Bovqh3HoZ9d_ambVFjRx2aN66pvtcrD3UptpwxFz3ce4hXnU8LNmdwa6jv_IJHMcMPiHTwgO6ONbvrIPLkpfDGt9gdTTBIhsfsqlabvqeCuCGSdFXqFK7he1fmYrDS_Bv0E3vhXgqKEvnKGdgKHYE3pOOAGKjqc_SOm6r-IZTSDDGyMWQpk9hHipbJw03_EU-UII_YoNL33ke7lDSIOL9im83gSefNzU3Eijwxp4ibHE9bSjZyhPoBo28gYOFgaBGg70rVGbSjQmdn-oyQLaeGWNSCQrRW25Or7cixR3NUY4JgJdnCiHDSGORn8aX-lyDKennhqUVQcGZltssQqtyvMScYBkPG8v_kk4FBSM4qslGt-Qw4ZCVyN9lAicP1k0D7SE3IjKf5jADEuUbtClnxA98UNYEkREwC2KX_AMfquBjdAxR9GyutpmDktOoDEifqAK1TBAbzwVv1xdMM32EGAG2nZ42RBRbNUTB0sTq
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.641Z'
      body: Good call! +1 to separate the RFC. This is the most riskful change that
        may be rejected by other vendor.&amp;lt;br&amp;gt;Nice!
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: Thanks!
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c15:
    status: RESOLVED
    token: AX5pRSnmMjrv1eeA6FVZLbqz_kUBSOdbn6My5N5RuXUKKNQFIdsVimtpDJdwR-CPTcVFS8m18GBJXi-6odIaQzpvFFlE3rRnp_fnV4TzXCsS0FnWEmEYOshiTf_H712sT-CVtIuMNrhEuM1fjxKTaLOujhKEYw8cIhYKBU6pPvFktrhxqosWCgnX9iY-pp7PZlrJJDQ0muR5orYE5O1dlDqKMhw17LW1E1RUOtFngXl08fjLTE9k6aecxLeXF-OkyWgDC9ZmIvGAYv9DuJ9wns3H43hiyXLg_BLUtF_iAetv97biDsePivFz_tKjviSfycLLkxUQaCrbsHvQsMmcV87Iib5ee06-KXBU5hls4K29cdy0rPwGQWYuhxxYO0D8UEF1U1mEAEL7GDdApHtwZzeHQafLv1EcDXHuwG2B-3amEffm9xPZKEfwRwKo2CZbt3x9TdYdL354SxzZ6klwYYUqfzBy_BeoucpiW0wbW_iILRU9KRs_WRyTK4nbuIc_xwAdIafTnm2DDOi5Y3PxAA9aU0ukRyw1kcP4Hov-jasrHW-x7VzPwgNDrR6oxmNDQAaCcVNz5mJf
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.641Z'
      body: Let&amp;#39;s mention that passing [] is treated as an error.
    replies:
    - author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.642Z'
      body: Also nice to talk about the error cases that this API rejects and how:&lt;br&gt;e.g.
        multiple file inputed while the API only accepts single
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c16:
    status: RESOLVED
    token: AX5pRSm2UE5Cf85-3iWiX5YdAG89R6yURqJy2HjpgmjzyMegqX_dYv2Il2QAH5b7yji6oYVPR91kgvdqh0KMZ8rhtqiKya5eXA8Lz-BOUOoE1JddIML-VVJXBKQ7fr10NdqOTiyiTswsThHhfWTLPH4DmUX9fkgpXIEUih7_BoP98TnECwy-vLDqNqGqFajNPafCyqGGzfPDZp1ndaoH7Z4xrf9wZX8-rSqJrNVPforp31o22yNBtDGYeXk-FfMBGUPEVMYPD8Tj66EOVt5KpyNSgf5S5HHUq39Cc2zEoHUET1QAwO20fYwQNNcWFVJZX28Yt57Px_G5sC8dwWnsaB6VROEPYLjqZxGE_kOAL76mJNO-RUhemkcgNFh75sO-IWC1hQ-KRydu8sadpuylZMy9OYY5UUeFjcqKh3cYwQ4ZTRPjJvFj1T56k3iwlu4f4nRMlkUCKFPYpjgjFetTlhv7ZaEWpe3HrSuxVPv01ER-dDi47IU8QDQ87famInsbGW-3KlmvP3NhzBba8nMK83Nnhqri125SyTpHG3SrlrnJaIBYczk722MWv08vSwAbUqaflGNfkLjYzGN1JcYSWejyUtoVlFCKiAt10e4r1ASYQ0-_3m5b_D-sKYBU3MKOu9N46JeMuA8NX890kgV92r6gP8SsdazSqeZ8T-4Z53HJk6_61tcAfclXvjl3cCYBSIC8vn96AWkrr_F9g9Pu-9kVN2ErxIhbmA44kbXw5w9TE7xq4vEINzP6bjHW3DzPikL8Nzf90q7srodBgKRLZyZnK4BX1nXLgGpZ9rXITq125D1KMMDKgNgjZS40LXWzvFiSTCEAb8u1f547OK32l0zOE1RRwE_I0BhaQR1lQqPT53MO2c_sSEmV7_gHL9h3UslBJQH7mv2ObaWofh0YCTCslcqjRtJ7VNNcig7a_Voawsd36tnARViAOZZe5Y1ozzoqpYru6GgbyBvK7Oxp_rz0jCDIjwzYAeRrNOfkCkKP4-LcNqk3r6KGDsQv-5aLxWyCtS6BJMPR6WMHt0nU8h1mfjR9ldL8grxv1M7QChS90YV12XSlCe6izGggmXrdZV5a2njUZ7Buu_PmWta9oAYdEO1kW-Ct-HTqjOguzbLv3kQHu93knWreG2XprZzBWQIcQVW4y2jzhNs9BQ6MGREEEXfjDgPUAuYvJb_2zLA6X-LSg3dbKCwW-UtdHTKU4APWFFj_pjTEZ_tUUkHnGN9cCJKZG5SZxShFKMhSHwflLKyhMYngBV6MrbeGpgpQKxQZBJQPl0t9uiSoPsqyGB9bEXF_Zqw4cKDKQG77f22KepWKTIhEHK7eiB3Dzp-TpcAlDDz7htsbPHUKvRMjO0Ls_epQPo4y0Otd5xrRUH-RNxpaXWRpKr6CxKAFC8-HEmwk9_RQ9yg6-KyrDPk_jXFJ1fTAdwyLQ9BWB4EZKk6JOk-uyfASXM48AN2K1AFlZ446V3WsWUVKExwfSoJ4GjAdJ_DaFE80jyMvA3SKgrF5ls2lzVJVldWNH8n6SgxTEx9RHzcyLGMXOBcK8mQHbIecQM1s-hxtDR10WTt1W18zOwqKPJGDgIUhhgwQ2qRwvxbGFaqQb9Z011YEyUtmd3gKL0tvlczRVpP3wz1NPSSTjDkA73EOtXUUU1LRHxqi2iEigMFGv9ub6JNmHt90YSGgLwtrCtauuBtC8RmQvTbf1BnOOxWNK0CUn7OplFfKlyyyx8c6qZZ3nBdDND69GO7UiQO5VPUhCvNnEzwjyKcKwW5i34qq_1vb85lda8C-xrGHzqmW9Jn-hqUkG5P_nkWthjmCIQ_9x7wDBpRPPGdP54kCiOt-Lq13sBMsGMechFXwGgGOZ781LO3-lLBjpTvd_aZ65E0xRUjtHxOy1bPkAczcxC1gZBOVbznNCpyvmhLMDvqUQGxRNPz1bdLhzg3AtyAtOj_vyHPFHQ
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.641Z'
      body: If we have a separate API for cancellation, we don&amp;amp;#39;t want
        to accept `null` here?
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: 'The current Chromium implementation accepts null under the hood (set_cancel_file_picker()
        is implemented by passing null).&lt;br&gt;&lt;br&gt;Keeping null accepted
        in set_file_picker_result(null) gives some flexibility—for example, in data-driven
        / parameterized tests where iterating over a test matrix with paths: null
        avoids an extra if/else branch.&lt;br&gt;&lt;br&gt;Do you think it&amp;#39;s
        better to strictly reject null (throw TypeError) to enforce using the separate
        cancellation API, or keep it allowed for flexibility while promoting set_cancel_file_picker()
        as the primary idiomatic way?'
    - author: Anna Sato
      time: '2026-10-16T04:51:47.643Z'
      body: Hmm I personally feel that if we allow set_file_picker_result(null), we
        probably don&amp;amp;#39;t need a separate set_cancel_file_picker() API at
        all.&amp;lt;br&amp;gt;&amp;lt;br&amp;gt;For parameterized tests, since a cancel
        test asserts `promise_rejects_dom` while a selection test inspects the returned
        handles, the test body needs an if/else or a separate promise_test anyway?
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.644Z'
      body: '@&amp;lt;a href=&amp;quot;mailto:&lt;a href=&quot;mailto:<a href="mailto:elkurin@google.com"
        target="_blank">elkurin@google.com</a>&quot; target=&quot;_blank&quot;&gt;<a
        href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;quot;
        target=&amp;quot;_blank&amp;quot;&amp;gt;&lt;a href=&quot;mailto:<a href="mailto:elkurin@google.com"
        target="_blank">elkurin@google.com</a>&quot; target=&quot;_blank&quot;&gt;<a
        href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;lt;/a&amp;gt;
        what do you think?'
    - author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.645Z'
      body: As chatted offline, I think having 2 different ways to do the same thing
        is not good, so we should decide which one to go. I don&amp;#39;t have a strong
        opinion on which to keep, so I&amp;#39;ll wait for your new rfc!
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.646Z'
      body: 'Thanks both! I&amp;#39;ve updated the RFC to unify them into a single
        API: test_driver.set_file_picker_behavior(options) ({ paths: [...] } to accept,
        and { action: &amp;quot;cancel&amp;quot; } to cancel), which avoids duplication
        and stays extensible for future options.'
    - time: '2026-10-16T04:51:47.647Z'
      action: RESOLVE
  c17:
    status: RESOLVED
    token: AX5pRSkOwm31UfFzmM5tEgITglspg7K7g7uduTNY9s8UdoG-2MiYSgAq_IpXhDco4I9CzR0n0VsHRdx14VhVLLIEIwJ0gkXeCC63GJb2_LEqKOpKsND9-0OgcmHQ7RjtgPJ03xmNHgcARccQHBfOsQYnTZIIP2wJ6eJw-zauSh-W6RTrsaj47Jss6Ae-EarsJL6IxHP5YRFYDOWdBtko-qbKJ4RfzEQ68xC5Fd4ygtUYWonQgG4e8khJpxTYtiGH-La7caKS0yPvq5gQ3TNNH_SR2HnZyszzGqQ0npmY_LjyTc4cs6ypBr1icjGTik5t3Q-LduiBrb0mkAVsvXiuU6N00PhltBDaL6p5AgtRW9clmCVnnO2GLFCNVrsWt-_1mPfRUUKXizwr5VmE6Bp_v4L-T_K4MqDmEF7-VGxmWcgQ58U6vg3fI-qIW8a-rVGRqpTdex0F9ZXF_K8JDnZ_wq4iQAIXV3i05FS0hksJUQsGOiyHYr4_rjrAE-zWpaNkehYZyTVfa1j5vt8tHIQ9PgJJt5-hNY8VIm8NOrF7EmDtx7l1kZ9oylbHHiHHC-olMA-uJYbMBabq8DYrVnbTL_wSUnIXAU8lLF8Hy4AzRhsKHBYQLygld4aWBq2tXpyDgBJr7BYG528
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.641Z'
      body: Thsi sounds like the result applies to every later picker. How about &amp;amp;amp;quot;the
        next file or directory picker&amp;amp;amp;quot; / &amp;amp;amp;quot;the next
        picker call&amp;amp;amp;quot; instead?
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: updated
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
    - time: '2026-10-16T04:51:47.644Z'
      action: REOPEN
    - time: '2026-10-16T04:51:47.645Z'
      action: RESOLVE
  c18:
    status: RESOLVED
    token: AX5pRSlR9mVsKNLAgk_eiTv5-F9_vh6MPDNse0asfCdu11EKgAtOHjCobcHqRI3JsDPPgoltuvUeJWzuQUqOyEBDJxSFGgEdsaI7Wd8slyb1gfv3SMzO3kqI_YGzeOMLbg_lxAJcV8jNdHhREeKpZIpk-T6x-SDsq_fx7yhQ6S0-8xiF4XfJrgPxLIrbctaKpxJOV0y9GxRr42GXBa4GkkqJYA7pkcdMjvTSjveoasK2OEBBfXnapcWDb_TBgGwEHu9crz7iFQsG_j-C6gnRMHuGdAIQ40t5a0179luUGz5vKOegKDffHTZyEg_MW_1gQEw-C3olmW2cig6OkXFRFmxbsCCkeprbdbwJce3o_ykcPEq5iKmk-CK7wJrydHLLjzww9mqQ0rlH6CdB0Q4s1T4kxPYseWyA254VazaJSTJklQ3dzja25nx8jfwg_crrRAHcugTy1WLGCSo56roEqSHCLE-J7lwr9CHLGWUGH9AMupij2xxwJjK-MF3WDilGXXtkFb1DenK0o52mjjHhjBzOTshM_EJ0cD49UgckGCqbuRcCxCcSmRLtMnyxT0gD1ZXQF059_NGBmM_enCQSn0tvjhd3eMxIFEXcwOXYbBNSywrcmsjMspZPdpR3f_w8DTPMWnnJDg6ggAhlEP2Z5iLns4BZtZt-e-Bc_9TvnnBtOBOPJBrb7YBAzzx8KpbPJ1X1mRGbLkw-CHvLnDvpWNT3nrKTjdmtqIiFBNNE__vum8WEkjR4j7a5MxHe8UPU8eRsMdNA0kL7fMfFDhY9C74IROQ1mUBwJ6C4BAe126JHFPDcJ8O1tMYLBDH7BO__7svv9OK5mJ9f6JDSVpVnEXB_IcpzV5RBy_B_2W0BhKMpdlMJ7LBQ1FnPW9MzQWdN5BYxke70vLjOZZrr5qb_zwW6RPKWwo3AdsXinBbUOzvR0OwoShm5T5fzmVsd4B3j88ayehLymrxclhISN8LkH0HUq9nKWeicYe6kbqsG6NjAu4uJGU6h7D0_vCmRo9MB-9S01ITng-FwZrO5vscEfcRUWmFiHRfdaWdo_48wymVa6p_AreNTRkb510ZSXhngX9htDxNHKkhJHsFlFOEGMPURrLn_ClrUp1-atTNQBINCiRTm7aT4FOXUN5u17rNblGYpiCvdPJnEzFvUrVj1KQidAvjxousWvqw5YuhHC-Nbsr9_IMcpBeCh5c2_664Adfs2_TlUsE0bTbNNY1pZfIv6cAujdSlOx2f804JxnLu9dk6GOaT-FiPSi7wx53VXRX3KjNCAPChp6kEWzl0t-qRvmIhzUBDtQL7LztVzursTLQzPT0luxzgzEOIE55OfSX2UfNgWTWWYNT0JVFfmn7-bZz7FFbKBSTHHkStbuxVWxdzHscgePTOtkZi1L1juvy5LB0jzm4mQwzLeUQN_ptd3PHDFYkP-24budPc2IO6o2C1ZLs3dYMzYV6RGGEo9Rx9oJns-3oaTTZuFo7rsR-_obc5ln4CMTg53Ig9TAhhUMOUaESu-BRJGJOotAg2463kal-7XMDUe4ezawRlTZWvP1LaUmjHPCRcVoZ7hQQG1nGOcXONvSBvE2PIuu4xUcuMBvme6djE64brsi5BLX59SBo6Sc16kUTu1XDlUBm0jetEjPaEqx092fRt4JkUk0XqIpjUIYHkhpy_M9ZX86kzaIN5I8pOv6TVDRKuX0yTsic3yWY30OYRa4Jh5t_OtyBcCM0GVDmP6oU9K7yep4Hawhpvi
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.641Z'
      body: If we have a separate API for cancellation, we don&amp;amp;amp;#39;t want
        to accept `null` here?
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: 'The current Chromium implementation accepts null under the hood (set_cancel_file_picker()
        is implemented by passing null).&amp;lt;br&amp;gt;&amp;lt;br&amp;gt;Keeping
        null accepted in set_file_picker_result(null) gives some flexibility—for example,
        in data-driven / parameterized tests where iterating over a test matrix with
        paths: null avoids an extra if/else branch.&amp;lt;br&amp;gt;&amp;lt;br&amp;gt;Do
        you think it&amp;amp;#39;s better to strictly reject null (throw TypeError)
        to enforce using the separate cancellation API, or keep it allowed for flexibility
        while promoting set_cancel_file_picker() as the primary idiomatic way?'
    - author: Anna Sato
      time: '2026-10-16T04:51:47.643Z'
      body: Hmm I personally feel that if we allow set_file_picker_result(null), we
        probably don&amp;amp;amp;#39;t need a separate set_cancel_file_picker() API
        at all.&amp;amp;lt;br&amp;amp;gt;&amp;amp;lt;br&amp;amp;gt;For parameterized
        tests, since a cancel test asserts `promise_rejects_dom` while a selection
        test inspects the returned handles, the test body needs an if/else or a separate
        promise_test anyway?
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.644Z'
      body: '@&amp;amp;lt;a href=&amp;amp;quot;mailto:&amp;lt;a href=&amp;quot;mailto:&lt;a
        href=&quot;mailto:<a href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&quot;
        target=&quot;_blank&quot;&gt;<a href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;quot;
        target=&amp;quot;_blank&amp;quot;&amp;gt;&lt;a href=&quot;mailto:<a href="mailto:elkurin@google.com"
        target="_blank">elkurin@google.com</a>&quot; target=&quot;_blank&quot;&gt;<a
        href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;lt;/a&amp;gt;&amp;amp;quot;
        target=&amp;amp;quot;_blank&amp;amp;quot;&amp;amp;gt;&amp;lt;a href=&amp;quot;mailto:&lt;a
        href=&quot;mailto:<a href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&quot;
        target=&quot;_blank&quot;&gt;<a href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;quot;
        target=&amp;quot;_blank&amp;quot;&amp;gt;&lt;a href=&quot;mailto:<a href="mailto:elkurin@google.com"
        target="_blank">elkurin@google.com</a>&quot; target=&quot;_blank&quot;&gt;<a
        href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;lt;/a&amp;gt;&amp;amp;lt;/a&amp;amp;gt;
        what do you think?'
    - author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.645Z'
      body: As chatted offline, I think having 2 different ways to do the same thing
        is not good, so we should decide which one to go. I don&amp;amp;#39;t have
        a strong opinion on which to keep, so I&amp;amp;#39;ll wait for your new rfc!
    - time: '2026-10-16T04:51:47.646Z'
      action: RESOLVE
  c19:
    status: RESOLVED
    token: AX5pRSm3l39jggGxZd03kcZQmxXawfwxjRUVuVvOO5uOVVOTIFx8wG6ttZe23BHQF_27ryMfLy1FalDu6s7w9ZVK13-I-9t1gbN6vnrGjfhzHqfABS7A7Rc8EGtUnZRaGvkYvhrd6UPX-lb_r65WlY46WMcq7fXQWuhrZ4yDvNfgKM9lSlUIAeBA3gZ98f8E4DEwlMVpUjan84QCrn0as22jZ7cX4SlqgnpGjkFuIUPyaZlLqESI1amftZfsAfXElq4USkZv4Bb2xHNX3_89YoKOW7kH5O_xJoYi3hB6WLALhxwAk1Ig5LWqUJReBM8CFvcZoKjW7k_mboLb437G5GZEz0dfnxUnYGYj_OuUceWqVPyT-BpiWqeuijIJVo9lg9sJ6JXf9c0MYxHSdVUIJZys5T8Rtcf5XoApPraV3uxUPzcmJjdPO3lNU-DS6Kl1YcMtPH4Aq2rwfOQY1eeyfoQE-1mmL9Lvv7aUN95aCWZkKnf9-S6WDxtjC6LzfONclTxvVKHfL0DtBAF9aQbYb8CQy_x9uznMAJ__yO7NctBcIm-ZX4clu_9rhyQWZID9t1hmQbwT4ZHuSK4Et611xfCBu3bLAoK2kvV1F3lja1ieKgIFVnBkHBioia1txKXTpYZ-UYD7OR2CTruG1bJ2ZH22Cmwa44HF34S18XMUxsMw6FWoDWTMjDt9lPQJXcyHM09YfFmNCKgONBAhUVN2ElR1EPPEkDjfWRlWd5CG669ewwjVkWKEPKbXiYsHsIzalyhkA5BMdt5VUWW8gGa4oAHVxIdgLCJXHY3MPw5BbMTW6rZL5PDVweThmsiY8AZy8ha6MShlqe8-hgK_BAXxObh_tHqa0Dw6WN_iersmFuKyfqsqF9KVEPQXRp5NMSml47th7srYZX4osbrhm-qaudVT4dJuhW2qHSs5xU9YCcM6WQeVRRAIcpFnbBU4fAkZF_ImYd--V1Xzl5J8qxMJOklzByB3LANWLLczqvF38EiobgRWeEUgP6QfTi2vIagqPisBQfuXBx1nIdNc9fHiuwcdrgz_8LAFZp_-2rfqo4iotUWp_Wv6dCoOM2eyEAjVrbCpXNY5v2Iw7Q9GO85iRUZ3-CLpBVIHX2NC7-bvDFpj3jKul6TqzWjrI7RT-VTblnxC_TxR5znjfdw5cxG3EcgDXuHlwncYoY02Sxbl4bzJ67rNPFwoaAUAJEvhf7OCt5i8_z7l4vtW26tJg-gZIjISI7FrY8OVSgyy7iF7mEAsyvOvVGocmrNeI3leBcKENvx4IsVUqX_aZ8ZCZxauG6fAHkt4kzpMgjmDZ1w5TCq2bvwtmKrA_sdRmKikZSLhHtVSC-HU8RAGhtm4JfpkZyIUugLupNdWmSax0rRdXkpalpFNOd6I4pr3t3VOaffGxwMdMkxne0tOjeQ43dQM9etqUE29yb5RkyAia8b_vxFC8Bch9YvODErZ964tEXc_mVQQTgBGPoT6rE-o0mO6hO_m3rMFG2IOAr7EjuGl5Ul0648ni3BDE0HzpWSQ-9AHb23ExJGNpqHo2FFBwUS75JScKOQBvrJRirv94VtFR_349WgGVqUMGCqDsQrl0lNvXNDs_HiYi4bVdMb_xeSd9dgGPF6fjwXG4wVdrZ354CBSYc8YfGQbutFcXl8mLutT2xVZ0FkAglq9YgbM0Xd7Ny-nm3s7slzsgEaLK8NymudE9uf8IiiXVQb731OodWhP26ib9LjkhFGQy6ChBnJXHS5yJHR7xhVuieC-4tl2-EXfkoiG0TOFBWD_IhK_Cg6Bx1dAMaaeCrkLx6Iq3k_ahyqfQ5UXUHm_gcV5oTxz1Jom5z1kMBpEbSr3SP0XCFyUEOqBDuCMthpzg7wOXXMcjtvLy_-v_CBzNTi8bSx0VvtzApHW0bJ8C9C99uAJmuQgEGUsBfIv-QOhMM8TZAcR8LAHC1HuIe4Bnt18lcZ8QZZNplcFjgLqREfMYJDWqlwQfEfqCiZyCuJ9gbLy2Flgo9Y1-oHg1g
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.641Z'
      body: If we have a separate API for cancellation, we don&amp;amp;amp;#39;t want
        to accept `null` here?
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: 'The current Chromium implementation accepts null under the hood (set_cancel_file_picker()
        is implemented by passing null).&amp;lt;br&amp;gt;&amp;lt;br&amp;gt;Keeping
        null accepted in set_file_picker_result(null) gives some flexibility—for example,
        in data-driven / parameterized tests where iterating over a test matrix with
        paths: null avoids an extra if/else branch.&amp;lt;br&amp;gt;&amp;lt;br&amp;gt;Do
        you think it&amp;amp;#39;s better to strictly reject null (throw TypeError)
        to enforce using the separate cancellation API, or keep it allowed for flexibility
        while promoting set_cancel_file_picker() as the primary idiomatic way?'
    - author: Anna Sato
      time: '2026-10-16T04:51:47.643Z'
      body: Hmm I personally feel that if we allow set_file_picker_result(null), we
        probably don&amp;amp;amp;#39;t need a separate set_cancel_file_picker() API
        at all.&amp;amp;lt;br&amp;amp;gt;&amp;amp;lt;br&amp;amp;gt;For parameterized
        tests, since a cancel test asserts `promise_rejects_dom` while a selection
        test inspects the returned handles, the test body needs an if/else or a separate
        promise_test anyway?
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.644Z'
      body: '@&amp;amp;lt;a href=&amp;amp;quot;mailto:&amp;lt;a href=&amp;quot;mailto:&lt;a
        href=&quot;mailto:<a href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&quot;
        target=&quot;_blank&quot;&gt;<a href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;quot;
        target=&amp;quot;_blank&amp;quot;&amp;gt;&lt;a href=&quot;mailto:<a href="mailto:elkurin@google.com"
        target="_blank">elkurin@google.com</a>&quot; target=&quot;_blank&quot;&gt;<a
        href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;lt;/a&amp;gt;&amp;amp;quot;
        target=&amp;amp;quot;_blank&amp;amp;quot;&amp;amp;gt;&amp;lt;a href=&amp;quot;mailto:&lt;a
        href=&quot;mailto:<a href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&quot;
        target=&quot;_blank&quot;&gt;<a href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;quot;
        target=&amp;quot;_blank&amp;quot;&amp;gt;&lt;a href=&quot;mailto:<a href="mailto:elkurin@google.com"
        target="_blank">elkurin@google.com</a>&quot; target=&quot;_blank&quot;&gt;<a
        href="mailto:elkurin@google.com" target="_blank">elkurin@google.com</a>&lt;/a&gt;&amp;lt;/a&amp;gt;&amp;amp;lt;/a&amp;amp;gt;
        what do you think?'
    - author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.645Z'
      body: As chatted offline, I think having 2 different ways to do the same thing
        is not good, so we should decide which one to go. I don&amp;amp;#39;t have
        a strong opinion on which to keep, so I&amp;amp;#39;ll wait for your new rfc!
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.646Z'
      body: 'Thanks both! I&amp;amp;#39;ve updated the RFC to unify them into a single
        API: test_driver.set_file_picker_behavior(options) ({ paths: [...] } to accept,
        and { action: &amp;amp;quot;cancel&amp;amp;quot; } to cancel), which avoids
        duplication and stays extensible for future options.'
    - time: '2026-10-16T04:51:47.647Z'
      action: RESOLVE
  c20:
    status: RESOLVED
    token: AX5pRSlnt7c4BwYbgA2R5nXqC0-xgpDBglf8ak35RwlD52e4xEOBdejvUKUBg3v50QCBtlFPgwQAS1_CAGzsLks-e-pE9jpPearHaw19OB7-rX2lm3dwHZIjonvQw2KmOa1omswPBMutHdYYpFoTcexBaAygL27MOqA5DL3EUmi6-MMTu1S968Aopy6M2t-iQku_qL7K_yds6T7w9f_L7XW2XRSKSpuw6UZ0Fy0cUSUk0qd4RnUR5gUw5JZBCdhzbMKr2OFjfvyQIF6AqxALYAEdeAjf3SUASrCRh0qUI_JW_CQdrzQGBcO0bi70uYf_omfcxbrAbSmiEFcim79LlJtm-_yNmWdV54hrfDJ8vrXF0erSnRzJ2epxw8UDN_ODlz95-XiAq_hRyXIXT8oxuXfkjtgtavbbAREjpcHuAC7bpxOHfJBeM_PZER0hsgpDn_LoZoZhH-sbb6gEmr9oYhK1
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.641Z'
      body: We should also mention that we need to introduce (or extend) fileDialogHandled-ish
        event
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: Done
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c21:
    status: RESOLVED
    token: AX5pRSkBQj4I4Bd5pzJ3XeYLTCAtQUu0WglNKFyA5Km3w22UNIcONH0rqmQnc91h4N7OSl5cR39_RRaq_YMPjx2jNrRl73IzyFw4xosa9QxOADzxKvciGFznAF8NpCoCCLCHZ5SsUBrZ3fgZGmY8V-AnE2UTwi7qw-5ER0XKOM8oGBUrVx4ZPixIRbPTze19cZLIhQogjsvsbyrfi31cTnlg68cEH_QZu0T55uE7cySCChDFGN1E91pW3zGBNyuuMhUDNXRSQBe02uAUtoIR8bUSZE1snC-NfakbK53eeGHAdOUAXPvaB12-gIZUDWzm75vfT9HllvK12Dqq0WExDfPlQW5m32nUP2RGbu0G8DyS4H20HNxtQ5bVHWm4rsIyw-b1qAcPaMZG61PmHdneoKUwuSuxcW674GfHqHh4lA6VBdDRxDbhR4GJ6X7p63GCNnXa5-XkAwbWdxyTkVMvHco
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.641Z'
      body: We should also mention that we need to introduce (or extend) fileDialogHandled-ish
        event
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: Done
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c22:
    status: RESOLVED
    token: AX5pRSlPu86shqospDHvZtJ9mF8mbeD-muBBRNfKVKeHC0PwMhCTp8w1KSTVmQipObPnw16GYjwUAA1Sem2PykByx_J5bLGGVIWDZXdpYXu5Y67-ASei7HvmKGzR_xs3cmPE_YKjZ6KZIPzHOFiE1pLzEnpxH3RyKq3eTg0rqdopNHozei6-_K85aUXXSM_qSnfekamzepjskD-7AAKeC-NniY_rHGWJcfFUwzmc1rPN1vUUCdK0PhUBwhRFAL0jhjjDpq5nsR-wSTXmkc7OTmnxQk-Zprmw10xNzF2pMZrPSMmMedGgn9qfMLY_CJZJMhp7GIwJ7vHIkUrTSVBALn1Id6J9hxD0rqdU4CVr_dsXnElVgCG_mZrU7cY0XEorpTmJahv23zrE_mXzXEyoTHRhMPC6kjHe9yjzFfQdI5PealAAXIU9ggrouoVRdmX5Eyt5x_2dH40VpWGJBxSfWR5DoCibWxOKH7MFLWm03Gz9QvOiAgwGqPVBiPVRTrvmj8A4nHY
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.641Z'
      body: Good call! +1 to separate the RFC. This is the most riskful change that
        may be rejected by other vendor.&lt;br&gt;Nice!
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: Thanks!
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c23:
    status: RESOLVED
    token: AX5pRSnHXrol53b_IzUXxunmSMDn6V5d9j-3oMqSNcl9jdVWGvMuXkg-RdWSnE4gpZzzjLtwIVNcYkrjp_JcIVc1oDdkTTipjdpj3DrXgM8UekdHiPjBYzlmI_auDOXFO3cQzwD3GfSvjeZmWE5bM0qb51HoGQwOgIgdGPLWeFToBx8LOhgmmlVyVpIRzvHM8Kl0Vn79IiUeE68IYpPrWLtKaN_joWuocOpfcAoOztRoDRcsHDoLyNMQx8YuC9a7bh1NJe2SG9Tf8LhTXTda_6FBwDJNT0TWRoe5w7VfAI9KzrEeZ3RjJoN66XCN33gBb3lQXWhshGfFh8HH_azvx-a0Ht4V5ZPpjXAKNj0vBPK063f3ljyrgqUeQiYz-bStPH5IPXGGUsvFqhLV0aS2ktGsEzTPFDSGj0YB5qLQojL5l_lK6HzvXJVgY13aLGH-sJyYfeVN_s63wbHWd1ShkzUfOA1FDajQ6bioNKGjIR4XFPvAfvqCDW2RQYFa2rlm3tgp1UcnUfGZZXwBzTMbair5GSU
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.641Z'
      body: Thsi sounds like the result applies to every later picker. How about &amp;amp;amp;quot;the
        next file or directory picker&amp;amp;amp;quot; / &amp;amp;amp;quot;the next
        picker call&amp;amp;amp;quot; instead?
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: updated
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c24:
    status: RESOLVED
    token: AX5pRSmUNpptzRNDjwkanFGWB8M1FVFwg1xF4XLv52VXBBbu7o8OCtEOROT9CjMgKU_doyaXoBRtj0GAQThm7O30ibWAXiYX9TIVNBQq33q2rkP7bU-Z5RTntEJZFQlxjp1jAodRLT7HEoddBYcUL7DTF7RCBeOCv86KO_uYamVL1nCxHPz6ES9sQcOwM-QgG-FQ8xhU59N-kBe4SXAqgHBLzB6ZFYVbihPVlvpWZz6nSoVyiM0-KyxtiqZPwi5dZBRbMZgPqB-XfVV_rdUfMNIV1ZqRZTfWsZZBnn6ir8x1UknD10_3JNkuY4TZGXXlb7xeQzPRYgPY9pqAaqZnylLc6Clx-OSvVEKxH_XsIsl9Nj3Czrk9cms0BNz1i_8_5hlME1zzKkhN72afTxUw0VKgZUSG9hsK1nwUt1wlblLBfFDvxqOfSKLHjboiwjeZ7GuTr1CuEFqRpqSL9PKJMhP1Qhj8SilIuiPBZJDup3uaph0JSkSMlTIhVj3isUVqg_lb6sk6lIeS2H6NxtsDLxoS-AKEfZA
    head:
      author: Anna Sato
      time: '2026-10-16T04:51:47.641Z'
      body: Thsi sounds like the result applies to every later picker. How about &amp;amp;quot;the
        next file or directory picker&amp;amp;quot; / &amp;amp;quot;the next picker
        call&amp;amp;quot; instead?
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:51:47.642Z'
      body: updated
    - time: '2026-10-16T04:51:47.643Z'
      action: RESOLVE
  c25:
    status: RESOLVED
    token: AX5pRSnuO0B-MvFRvHriSrpLPmXcMyiELJ6ysxn519kbxNUE0Gcr0ue7GNDRB4x8yWvMC5dssGAhtdnhP5SjF-onK4A1E2kdFC-n6l0sFkSJIpNzijzpZHsOHkrrYc6f3r0DxF-_BnThSRrhTpFIqkmZKejnkyJkn9q3VQ6JplDqbZ8Q5KSJFRyQSiUFVjuHDhcXOvjZjVdTVsG5bUv3bWdgmTUEvpcQuKldyC0olpPFjiO0-n3v1CPkZE0bZNHO0HW2QGljCa74VQtqZOM85EstIjSsL00fmADVXqY92fLitYyZIwZAnr6HDDkGNgBlRfAX_N4w4lAEOhQU-tB5nX935oiqvSUn1XtYKTdA3PpywfCfBRL3ax5PxSYdEOOg-WeBvi0FKScMXe1k6Z9Ota3Ncaik-9ztzIArbw_CRvqJCwIEqbixQ8l-jbNHC0J7V_RUUjeGNg_Gy77ODS5SfVmC7ADvDDiL54viLh9pkd43OWmggGfHjazOZ0QktcFPelTDIxQdARLJOFUtXsGLq0oNaot_0Xo9Kg8LDbbofu5ORUKvdh5XHInN-aHl4QmTvvaElRW4jWX19tmB1S0YScOOKUuiilN8qaOo3pxf7jUNwoJXeKeBiwLZlhAVb25Vas3tyUWMlJW4-aA
    head:
      author: Eriko Kurimoto
      time: '2026-10-16T04:51:47.632Z'
      body: We should update the title too. This RFC is no longer limited to testdriver
        support. <br><br>Maybe:<br>File and directory picker automation in WPT
      quote: Add testdriver.js support for file and directory pickers
    replies:
    - author: Ayu Goto
      time: '2026-10-16T04:53:28.760Z'
      body: Sure
    - author: Ayu Goto
      time: '2026-10-16T04:53:31.623Z'
      action: RESOLVE
---
