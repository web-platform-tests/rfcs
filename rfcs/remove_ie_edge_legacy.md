# RFC 179: Remove support for Internet Explorer and Microsoft Edge Legacy

### Summary

Remove support for running WPT tests in Internet Explorer and Microsoft Edge
Legacy.

### Details

Support for the Microsoft Edge Legacy desktop application ended on March 9,
2021[1], and support for the Internet Explorer desktop application ended on
June 15, 2022[2].

The proposal is to remove the `edge`, `edge_webdriver`, and `ie` options from
the product parameter to `wpt run`, along with all related code.

After removal, the remaining supported products would be:

```
[--product {android_weblayer,android_webview,chrome,chrome_android,chrome_ios,
chromium,content_shell,edgechromium,firefox,firefox_android,safari,sauce,servo,
servodriver,opera,webkit,webkitgtk_minibrowser,wktr,epiphany,ladybird}]
```

[1] https://blogs.windows.com/msedgedev/2021/03/09/microsoft-edge-legacy-end-of-support/

[2] https://blogs.windows.com/windowsexperience/2022/06/15/internet-explorer-11-has-retired-and-is-officially-out-of-support-what-you-need-to-know/


### Risks

* Someone running WPT tests against these browsers would now be broken.