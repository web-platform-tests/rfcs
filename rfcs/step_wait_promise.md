# RFC 207: `step_wait_promise` method

## Summary

Add a `step_wait_promise(promise, description, timeout=default_timeout)` method to 
the wpt `Test` object. This method waits for the given `promise` to resolve within 
the specified `timeout`. If the promise is not resolved within the timeout period, it 
raises an assertion error: `Timed out waiting on condition`.

## Details

* **`step_wait_promise(promise, description, timeout=default_timeout)`**
    * `promise`:  The Promise object to wait for.
    * `description`: Error message to add to assert in case of failure.
    * `timeout`: Timeout in ms. This is multiplied by the global
      `timeout_multiplier`. Defaults to `step_wait_*` default of 3000ms.
    * Behavior:
        * The function waits for the given `promise` to resolve.
        * If the `promise` resolves within the `timeout`, the function returns the
          resolved value of the promise.
        * If the `promise` is rejected within the `timeout`, an `AssertionError` is
          raised with the rejection reason, `description` and stacktrace.
        * If the `promise` does not resolve within the `timeout`, an `AssertionError`
          is raised with the "Timed out waiting on condition", `description` and
          stacktrace.

## Motivation

* Support for BiDi events: Facilitates testing scenarios involving BiDi events
  in the testdriver API, where asynchronous events are expected.
* Improved efficiency compared to `step_wait`: Eliminates the need for periodic
  checks with intervals, allowing the test to proceed immediately once the awaited
  event occurs, potentially leading to faster test execution. Even though interval
  of
  100ms seems to be small enough, it can still accumulate to a significant amount of
  time.

## Examples

In [console/console-count-logging.html](https://github.com/web-platform-tests/wpt/blob/ea3f611e8d3e7debc3b9cb98b0e63254657fa7eb/console/console-count-logging.html#L33),
`log_entries_promise` is awaited without a wrapper, which in case of not having the
required events within the test timeout, the test will fail with a generic timeout
hiding the actual step and stacktrace.

### Before:

```javascript
// Wait for the log entries to be added.
const log_entries = await log_entries_promise;
```

In case of not having the required events, the test will crash with a generic
timeout error:

```
/console/console-count-logging.html
  TIMEOUT Console count method default parameter should work - Test timed out
  TIMEOUT /console/console-count-logging.html
```

### After

```javascript
// Wait for the log entries to be added.
const log_entries = await test.step_wait_promise(log_entries_promise,
    "Wait for the log entries to be added.");
```

In case of not having the required events, the test will crash with a generic
timeout error:

```
/console/console-count-logging.html
  FAIL Console count method default parameter should work - step_wait_promise: Wait for the log entries to be added. Timed out waiting on condition
    at async Test.<anonymous> (http://web-platform.test:8000/console/console-count-logging.html:33:29)
```

## Risks

* Adding API surface: Increases the complexity of the testing framework and requires
  ongoing maintenance.

## Considerations

* The implementation should respect the "timeout multiplier" setting to ensure
  consistency with other timeout-related behaviors in the testing framework and to
  work
  correctly in the presence of slow or overloaded systems.
