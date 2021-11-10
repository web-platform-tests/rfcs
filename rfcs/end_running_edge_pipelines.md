#RFC ADDNUMBER

## Summary
We at Microsoft are considering disabling Microsoft Edge runs on Windows as part of this WPT.FYI effort. 

## Details
When Edge was built as EdgeHTML, there were very real concerns about interoperability with Chrome, Firefox, and Safari. However, now that Edge is a Chromium based browser, the value we get from evaluating runs has been reduced to small differences due to merge issues as well as some differences between Windows and Linux.

Now that we have shipped Edge Stable to Linux, what we would prefer to have happen is a change to begin a more "apples to apples" interop test by running Edge on Linux along with Chrome on Linux. An alternative would be to run Chromium on Linux, but we're not convinced that would return the same value to web developers because it would not expose places in the platform where browsers might be intentionally different.

## Risks
One of the biggest risks is that we will miss some large interop gap present on Windows that is not present on Linux. However, that is mitigated by the browser vendors running WPT in their own internal CI/CD systems where they catch regressions. In addition, any large gap like that would be present in both Edge and Chrome and would likely not present an interop issue in the web platform, but rather in the platforms themselves.

Another risk is that we will miss small bugs introduced through subtle interaction with the different platforms, but here again, these would not be defined as browser interop issues and would likely be caught in CI/CD. In the two years we've been conducting this experiment, we have not seen anything like that come through wpt.fyi.
