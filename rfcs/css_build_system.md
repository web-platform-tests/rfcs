# RFC 136: Remove the CSS build system

## Summary

Remove CSS build system in `css/tools/` and the path lint rules for `css/` that support that build system.

## Details

http://test.csswg.org/suites/ hasn't been updated in some time, which led to a [discussion](https://github.com/w3c/csswg-drafts/issues/6896) in the CSS Working Group about how to fix it. Ultimately, the CSS Working Group [resolved](https://github.com/w3c/csswg-drafts/issues/6896#issuecomment-1499457107) to remove the now unused CSS build system, and this RFC implements that decision.

These lint rules will be removed:

- CSS-COLLIDING-TEST-NAME ("The filename ... in the ... testsuite is shared by: ...")
- CSS-COLLIDING-REF-NAME ("The filename ... is shared by: ...")
- CSS-COLLIDING-SUPPORT-NAME ("The filename ... is shared by: ...")
- SUPPORT-WRONG-DIR ("Support file not in support directory")

The MISSING-LINK ("Testcase file must have a link to a spec") rule which also applies only to `css/` will _not_ be removed, as it isn't only used by the build system, but is also a matter of style and preference. This may be revisited later.

## Risks

Reviving http://test.csswg.org/suites/ becomes harder with this removal, so if that system has important functionality, it will require more work to meet those needs again. However, the web-platform-tests project remains open to contributions to support missing use cases, should the need arise.
