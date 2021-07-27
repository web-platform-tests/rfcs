# RFC 89: `testdriver` Add an `execute_script` method.

## Summary

Add an `execute_script` method to testdriver that takes a js function,
arguments to pass to the function, and the context id to execute it it
in a remote context. If the function returns a promise we return the
resolved value of the promise, otherwise we return the return value of
the function.

## Details

Sometimes a test window wants to check the status of something in a
remote window. It's possible to do this in a number of ways, for
example by posting messages from the remote window to the test
window. However in cross-origin cases messge passing on the js side
can be difficult or impossible. In addition when tests are defined in
the test window, it's useful to avoid putting complex logic in other
browsing contexts because these won't have the same error handling
properties as the test window.

A simple way to solve these problems is to provide a mechansim to
execute script in a different context. This isn't possible using
content APIs alone, but is possible in WebDriver, and we can reuse
that to define a `execute_script` function that works across contexts:

```
execute_script(fn, args, context)
```

`body` is a js function for the script to execute. To pass it to the
remote context, this is converted to a string using
`toString()`. `args` is null or an array of arguments to pass to the
function on the remote side. Arguments are passed as JSON. `context`
is an object that can be resolved as a testdriver context id to
identify the remote context (see [RFC
88](https://github.com/web-platform-tests/rfcs/pull/88). If the return
value of the function when executed in the remote context is a promise
the promise returned by `execute_script` resolves to the resolved
value of that promise. Otherwise the `execute_script` promise resolves
to the return value of the function.

The wptrunner implementation uses [Execute Async
Script](https://w3c.github.io/webdriver/#execute-async-script). We
wrap the provided script text and arguments into an `Execute Async
Script` call as:

```
let callback = arguments[arguments.length - 1];
let rv = ({function_string}).apply(null, {json.dumps(args)});
return Promise.resolve(rv).then(callback)
``

## Example

```
<iframe src="child.html"></iframe>
<script>
setup({single_test: true});
onload = {
  let value = test_driver.execute_script(async (elemId) => {
  await new Promise(resolve => onload(resolve))
  return document.getElementById(elemId).textContent
}, ["test"], iframe.contentWindow);
  assert_equals(value, "PASS");
  done();
}
</script>
```

## Risks

WebDriver is single-threaded, so all the WebDriver-based actions need
to be queued. That means that waiting for a promise in a remote
context blocks all testdriver functionality that uses WebDriver, and
also blocks the harness from returning results. This implies a high
risk of timeouts.

## References

[PR 29803](https://github.com/web-platform-tests/wpt/pull/29803)
contains a prototype implementation of this.
