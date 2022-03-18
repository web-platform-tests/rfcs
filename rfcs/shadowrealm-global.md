# RFC 107: Add support for Shadow Realms as Javascript global

## Summary

[Shadow realms](https://github.com/tc39/proposal-shadowrealm) are a new
sandboxing primitive, currently a stage 3 proposal in TC39 and in the process of
being integrated into relevant Web specifications; part of this extends some
pure/non-rendering/purely computational Web interfaces to shadow realms (which
otherwise do not have browser APIs and more closely resemble a vanilla JS shell
environment.) 

I think it makes sense to extend WPT tests for these interfaces to
also run in shadow realm contexts, so we can be assured they still work there,
therefore ...

This RFC proposes adding a new valid global for `.any.js` tests, `shadowrealm`
that runs the test harness in a shadow realm under a regular window environment.
   
##  Details
  
To accomplish this, I add a new `ShadowRealmTestEnvironment` to `testharness.js`
that reuses much of the existing machinery to run tests remotely in e.g.
dedicated workers. There are however, two main differences:

1. Shadow realms are not DOM contexts and so have no onload or similar event for
   us to use to actually begin test aggregation; instead, I add a new hook for
   the incubating realm to directly call to signal that all desired tests have
   been loaded.

2. The actual message port used to communicate test results back to a
   RemoteContext requires JSON-serialization of the mesasges (primitive values
   and callables may cross between a shadow- and incubating realm, but not
   arbitrary objects): it happens that this works correctly given the current
   encodings of `Test`, harness state, etc. seem to be JSON-round-trippable, but
   this would become a hard requirement from this point forward.
   
A proposed patch is available at https://github.com/web-platform-tests/wpt/pull/33162
   
## Risks

There's two concerns I have with the current proposed implementation:

1. It's unclear (to me) given the current state of the way Shadow Realms are
   integrated into the HTML spec how we might be able to deal with unhandled
   promise rejections. It happens, for now, that they will be handed by the
   incubating Window's `unhandledrejection` handler, which is installed by the
   parent test harness, and prints to the console. This seems acceptable for the
   time being, but may need to change as the spec is clarified.
   
2. Detecting whether the test harness is being run in a shadow realm is not
   straightforward--Shadow realm global objects are, by design, bare-bones.
   Their prototype is `Object.prototype` and contain only a handful of Web
   interfaces; the patch above implements a crude test by checking for the
   presence of `AbortController` if `globalThis` has been determined not to be a
   window or worker; if we do not require compatibility with e.g. nodejs, this
   seems fine, if, perhaps, fragile. A mechanism to pass information about the
   environment into the testharness early in the lifecycle would be needed to
   avoid this.
