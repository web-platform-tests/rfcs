# RFC 63

## Summary
Support running testdriver commands in browsing contexts other than
the test browsing context, including cross origin contexts where it's
possible to get a handle to the browsing context containing the test
(i.e. excluding the rel=noopener case).

## Details

For contexts in which it's possible to get a handle to the context
containing testharness.js ("the test context"), we can support
testdriver interactions by using postMessage to the top-level window
with a message containing the command to run and the context to run it
in. That approach works with the WebDriver-based implementation in
wptrunner where WebDriver can only address a single frame at a time,
and a WebDriver-driven event loop in the test context is used to
invoke testdriver actions and also to report the test results.

The specific proposed changes are:

* Allow testdriver.js to be loaded in contexts other than the test
  context.

* In such contexts require use of testdriver to first call the
  `test_driver.set_test_context()` function with a handle to the test
  context.

* Also provide a `message_test` function as a wrapper around
  postMessge to the test context.

* Add an optional `context` argument to test_driver commands that
  don't depend on a provided element, to specify the context in which
  they should run.

The implementation details for wptrunner are as follows:

* testdriver events are given a command id and a context argument. The
  command id is used to match the response from webdriver to the
  specific request. The context id is used to identify which frame or
  window should be used for the request.

* Ideally the context id would be the wptrunner window/frame handle
  for the window or frame. But this is poorly supported so instead we
  put a `__wptrunner_id` property on the window object, set it to a
  uuid, and do a depth-first search for a frame with that id in the
  wptrunner code when looking for the context in which to run the
  command.

* WebDriver is switched to the relevant context for the duration of
  the action and switched back to the test context once the action is
  complete.

## Risks

Adding optional arguments to functions in js is possibly confusing API
design and hard to extend. It might be better to use an options object
as the final argument, but this is arguably harder to use.

Searching through all contexts is likely slow and could have side
effects if it e.g. changes focus. It's unclear how to do better than
this without fixing the underlying WebDriver issue across multiple
implementations.

It is likely surprising to users that this would not work in all
cases, and by addressing this problem in this was we are putting off
the work to address it in a way that works for any browsing context.

Some of the proposed API additions are likely unnecessary if we later
update the implementation to work in all contexts.
