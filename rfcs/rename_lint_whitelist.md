# RFC 51: Rename lint.whitelist to lint.ignore

## Summary

Rename lint.whitelist to lint.ignore, to accurately reflect the purpose of the
list and avoid using 'white' as a synomnym for 'good'. See the [WHATWG style
guide](https://whatwg.org/style-guide#tone) note and the similar [Google
documentation style
guide](https://developers.google.com/style/word-list#blacklist).

## Details

Primary change is just changing `lint.whitelist` in `tools/lint/lint.py` to
`lint.ignore`. There are also some documentation changes to be done.

## Risks

Known risks:

1. Downstream consumers or infrastructure may rely on the name of our lint
   file. The level of risk here is unknown, but not expected to be high - it
   seems unlikely the WPT lint is a critical file.
1. There may be some mental overhead in people adapting to the new name.
