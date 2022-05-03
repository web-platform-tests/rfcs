# RFC #44: Testing origin policy

Note: This was reverted in [RFC #113](https://github.com/web-platform-tests/rfcs/pull/113).

## Summary

[Origin policy](https://wicg.github.io/origin-policy/) is a proposed web platform feature that applies origin-wide settings. Writing web platform tests for origin policy is complicated, because:

- The origin policy is a per-origin resource, located at `/.well-known/origin-policy`. To test different origin policy manifests, a test needs different responses to be served from that resource.
- Origin policy settings apply origin-wide, which can cause interference with other tests running on that origin. For example, a CSP that disables image loading would interfere with any image-loading tests running on that origin.

This proposal introduces new subdomains to the web platform tests serving infrastructure which are used exclusively for origin policy tests, and also serves `/.well-known/origin-policy` with the Python script handler instead of the static file handler, to allow varying responses.

## Details

An implementation of this proposal can be seen in [web-platform-tests/wpt#21705](https://github.com/web-platform-tests/wpt/pull/21705).

The added subdomains are of the form `opX` for `X` from 1 through 99. We are hopeful that 99 subdomains will be enough for writing all the necessary origin policy tests; if this ends up being too few, we can do a future RFC to increase the number.

The added subdomains do not contribute to the "product" subdomains (of the form `www.www1`, etc.) which were introduced in [web-platform-tests/wpt#13272](https://github.com/web-platform-tests/wpt/pull/13272).

The use of the Python script handler for `/.well-known/origin-policy` allows test writers to drop a Python script in that location which serves up different origin policies depending on the subdomain (e.g. by examining `request.url_parts.hostname`).

## Risks

The main risk here is the same as with any infrastructure change, in that it requires downstream consumers to be aware of the change and pick it up.

To the extent embedders are using `wptserve`, they will recieve the Python script handler change without difficulty. The subdomain configuration will be automatic as long as consumers are either using `wpt make-hosts-file`, or are mapping all subdomains of `.test` uniformly (like Chromium does with its `--host-resolver-rules`).
