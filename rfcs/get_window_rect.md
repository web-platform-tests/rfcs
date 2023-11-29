# RFC 174: Add testdriver.js support for the Get Window Rect command

## Summary

Add testdriver.js support for the [WebDdriver 'Get Window Rect' command](https://www.w3.org/TR/webdriver/#get-window-rect).

## Details

There is a proposal (see the CSSOM [issue 7693](https://github.com/w3c/csswg-drafts/issues/7693) for
details) of a new [WindowEventHandler](https://html.spec.whatwg.org/multipage/webappapis.html#windoweventhandlers)
element `onmove` to detect when the position of a Window object changes.

Tests for this new feature would benefit of the already implemented [WebDriver 'Set Window Rect' command](https://www.w3.org/TR/webdriver/#set-window-rect)
but we would need a way to verify the window has been moved to the right position. The `Get Window Rect`
command is already [supported](https://wpt.fyi/results/webdriver/tests/classic/get_window_rect?label=master&label=experimental&aligned&q=%2Fwebdriver%2Ftests%2Fclassic%2Fget_window_rect%2Fget.py)
for all WebDRiver implementors, but we lack support in the testdriver.js executors to be able to use it in actual tests.
