# RFC 173 - `@page` Rules in Print Tests

[RFC 41](https://github.com/web-platform-tests/rfcs/blob/master/rfcs/print_test.md) added support for print reftests. It specified a fixed page size of "5 inches by 3 inches with 0.5 inch margins on all sides".

CSS allows modifying the size and margins of paginated media using an [`@page` rule](https://developer.mozilla.org/en-US/docs/Web/CSS/@page). Per the letter of the previous RFC these rules would be ignored for wpt tests, making the rule itself impossible to test.

To enable testing different paper sizes and margins, this RFC amends the print reftest support, so that where an `@page` rule is specififed in a test or reference file, it overrides the default paper size when generating the paginated rendering of the document.
