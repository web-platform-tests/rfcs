# RFC 158: Variant name should be a non-zero length string

## Summary

Make a policy change to disallow zero length Variant names.

## Details

A test file can have multiple variants by including meta elements.
Even though zero-length variant names are allowed currently, we
can not run it the same way as other variants.

For example, the command below runs `websockets/binary/005.html?wss`
only.
```
wpt run ... websockets/binary/005.html?wss
```

But the commands below run all three variants in the file.
```
wpt run ... websockets/binary/005.html
wpt run ... websockets/binary/005.html?
```

This consequently causes problem when we do external sharding, then
pass all the tests we want to run to wptrunner through `--include`.
While we only intended to run the empty string variant by passing
`websockets/binary/005.html`, wptrunner runs all the variants in
that file, causing the non-empty variants to run twice. It is
possible we shard all the variants from the same file to the same
shard, this could load the different shards unevenly, and increase
the perceived total run time.

Proposal is to disallow zero length variant names.

### Implementations

Below are the changes required to make this happen:
1. Update all the existing tests that have an empty variant name. Update
the supporting code if necessary to make sure the tests continue to
work.
2. Add a lint to `wpt lint` to make it an error for zero length variant
names.
3. Update the document to not use zero length variant names as examples.
4. Update manifest generation code to throw an error for empty variant
names.

### Alternatives Considered

It is possible we update wptrunner to accept `path/to/test.html?` as the
url for the empty name variants, but this requires wptrunner side change
and it is not user friendly.

Alternatively we can add `/` at the beginning of a test name, to denote
the parameter is an url instead of a path. This deviates further from
how developers are running tests now, and it will break the command line
auto-completion feature.

## Risks

As this is adding restrictions to the existing system, it should not break
anything.

Developers should be able to run the tests the same way as before, with
the added capability to run the previous empty name variant alone.
