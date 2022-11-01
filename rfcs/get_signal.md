# RFC 115: Add a new test method that returns AbortSignal

## Summary

Add a new test method called `get_signal` to provide AbortSignal that aborts
when the test finishes.

This method will throw if `AbortController` is not available.

## Details

Modern web APIs have adopted AbortSignal as a standardized cleanup method. Providing AbortSignal can save some lines to ease writing tests.

### Existing way to add and cleanup an event listener

```js
promise_test(t => {
  const events = ["mousedown", "mouseup", "click"];

  const gotEvents = [];
  const listenerFn = t.step_func(event => {
    gotEvents.push(event.type);
  });
  for (const event of events) {
    target.addEventListener(event, listenerFn, { once: true });
  }
  t.add_cleanup(() => {
    for (const event of events) {
      target.removeEventListener(event, listenerFn)
    }
  });

  await test_driver.click(target);

  assert_array_equals(gotEvents, events);
});
```

### With `AbortSignal`

```js
promise_test(t => {
  const controller = new AbortController();
  const { signal } = controller;
  t.add_cleanup(() => controller.abort());

  const events = ["mousedown", "mouseup", "click"];

  const gotEvents = [];
  for (const event of events) {
    target.addEventListener(event, t.step_func(event => {
      gotEvents.push(event.type);
    }), { once: true, signal });
  }

  await test_driver.click(target);

  assert_array_equals(gotEvents, events);
});
```

### With `t.get_signal()`

```js
promise_test(t => {
  const { signal } = t.get_signal();

  const events = ["mousedown", "mouseup", "click"];

  const gotEvents = [];
  for (const event of events) {
    target.addEventListener(event, t.step_func(event => {
      gotEvents.push(event.type);
    }), { once: true, signal });
  }

  await test_driver.click(target);

  assert_array_equals(gotEvents, events);
});
```

## Risks

Some people may end up using `t.get_signal().onabort` to add cleanup functions.
This shouldn't be problematic if it's synchronous, but async cleanup functions
should use the existing `t.add_cleanup()` instead to actually wait for them to
finish.
