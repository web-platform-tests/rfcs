# RFC 51: Rename lint.whitelist to lint.ignore

## Summary

Rename lint.whitelist to lint.ignore, to accurately reflect the purpose of the
list and avoid using 'white' as a synomnym for 'good'. See the [WHATWG style
guide](https://whatwg.org/style-guide#tone) note and the similar [Google
documentation style
guide](https://developers.google.com/style/word-list#blacklist).

## Details

Aside from renaming the file, `tools/lint/lint.py` will need minor
modifications to look for the correct default file, and the
[web-platform-tests.org](https://web-platform-tests.org) documentation will
need updating.

## Risks

Known risks:

1. Downstream consumers or infrastructure may rely on the name of our lint
   file. Neither Gecko and Chromium appear to rely on it, and so the risk here
   is low.
1. There may be some mental overhead in people adapting to the new name.
