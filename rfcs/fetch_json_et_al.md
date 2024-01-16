# RFC 177: `fetch_json` and other `ShadowRealm` facilities

## Summary

This follows up [RFC 107][rfc-107].
[`ShadowRealm`s][proposal] are a new sandboxing primitive, currently in
the process of being integrated into relevant Web specifications.
`ShadowRealm` environments resemble vanilla JS shell environments, but
do have some Web interfaces available, such as `URL`.

I've submitted a [pull request][pr] with some changes to the
test harness.
The goal is to more easily allow existing WPT test files to run in
`ShadowRealm` contexts, without writing separate duplicated tests for
`ShadowRealm`.

The most notable change is a `fetch_json` function to allow importing
test data from JSON files across the sandboxing boundary.

## Details

The [pull request][pr] comprises four commits, which I'll explain in
detail in this section.

### Document self.GLOBAL.isShadowRealm()

This commit is just an improvement to the documentation.

### Add fetch_json() test harness API

A common pattern for importing additional test data from a JSON file
looks like this:

```js
promise_test(() => fetch("resources/mydata.json").then(res => res.json()).then(run_tests), "Loading data…");
```

`ShadowRealm` scopes do not have the `fetch()` API available.
Additionally, the sandboxing only allows primitive or callable values
to be passed into or out of the sandbox, not other objects.
Therefore, this approach isn't feasible for `ShadowRealm` scopes.

The proposed `fetch_json()`, in non-`ShadowRealm` scopes, is just a
shorthand for the above: call `fetch()`, and then call the returned
`Response` object's `json()` method.

Inside `ShadowRealm` scopes, it is overwritten by a function that calls
the host realm's `fetch()`, passes the response in string form across
the sandbox boundary, and then inside the sandbox turns it into an
object using `JSON.parse()`.

The previous pattern described above will continue to work in
non-`ShadowRealm` scopes, but if a test is to be executed inside a
`ShadowRealm` scope, it will need to change to a pattern like the
following:

```js
promise_test(() => fetch_json("resources/mydata.json").then(run_tests), "Loading data…");
```

### Expose location.search in ShadowRealm scopes

Browser-specifc APIs such as `location` are not available in
`ShadowRealm` scopes.
However, the [mechanism for variant tests][variant] relies on
`location.search`.

This proposed change would expose a fake `location.search`, with the
same value as the host realm's `location.search`, inside `ShadowRealm`
scopes.
This would allow test variants to be executed in `ShadowRealm` scopes.

### Use fake setTimeout in ShadowRealm scopes

`ShadowRealm` scopes also do not have timer-related APIs such as
`setTimeout` and friends.
However, several facilities from `testharness.js`, namely
[`Test.step_timeout`][step_timeout], [`Test.step_wait`][step_wait], and
[`Test.step_wait_func`][step_wait_func], rely on the global `setTimeout`
being present.

This proposed change adds a fallback version of `setTimeout` which is
used by the test harness when the global one is not present, i.e. in
`ShadowRealm` scopes.
The fallback is not exposed directly to test code.

## Risks

The documentation commit carries no risk.

The `fetch_json()` API doesn't change how any existing APIs work.
Existing tests would have to opt-in to it by replacing their use of
`fetch()` to import JSON test data, with `fetch_json()`.
A natural time to make this change is when enabling a test to run in a
`ShadowRealm` global scope; otherwise the test won't pass.
Therefore, I think the potential risk is low.

Of the four proposed changes, the fake `location.search` has the biggest
potential risk.
It's possible that future `ShadowRealm`-scope tests might depend on the
`location` object not being present.
This seems acceptable for the time being.
But if that changes, the variant mechanism in files such as
`common/subset-tests.js` and `common/subset-tests-by-key.js` would need
to be rewritten to indicate which variant to run using a different
mechanism than the URL search parameters.

The fake `setTimeout` should not affect any existing tests.
In non-`ShadowRealm` scopes, the behaviour should remain the same,
because `setTimeout` is available.
In `ShadowRealm` scopes, use of test harness APIs such as `step_timeout`
would previously fail due to the missing `setTimeout`; now they can be
used.
Therefore, I think the potential risk is low.

[rfc-107]: https://github.com/web-platform-tests/rfcs/blob/master/rfcs/shadowrealm-global.md
[proposal]: https://github.com/tc39/proposal-shadowrealm
[pr]: https://github.com/web-platform-tests/wpt/pull/43639
[variant]: https://web-platform-tests.org/writing-tests/testharness.html#variants
[step_timeout]: https://web-platform-tests.org/writing-tests/testharness-api.html#Test.step_timeout
[step_wait]: https://web-platform-tests.org/writing-tests/testharness-api.html#Test.step_wait
[step_wait_func]: https://web-platform-tests.org/writing-tests/testharness-api.html#Test.step_wait_func
