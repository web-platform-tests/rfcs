# RFC 95: End support for Python 3.6 on macOS and Windows

## Summary

[Support for Python 3.6](https://devguide.python.org/#status-of-python-branches)
ends on 2021-12-23. As the first step of ending support for Python 3.6 in
web-platform-tests, stop supporting it on macOS and Windows.

Ending support for Python 3.6 on Linux/Taskcluster will be proposed in a later
RFC.

## Details

The Azure Pipelines configuration for macOS and Windows will be upgraded from
Python 3.6 to 3.7.

The immediate motivation for this is that Azure Pipelines no longer supports
Python 3.6 on macOS 11, meaning that we will no longer be able to easily test
Python 3.6 on macOS after upgrading. We also use Azure Pipelines for Windows, so
in order to use the same Python version everywhere on Azure Pipelines, we will
upgrade to Python 3.7 on Windows too.

## Risks

Some downstream users of WPT may still be using Python 3.6 on macOS or Windows.

TODO, confirm with:

- [ ] Chromium
- [ ] Gecko
- [ ] WebKit
