# RFC XX: New Manifest Format (path trie)

## Summary

We propose restructuring the manifest to be a trie of path segments, rather than the status-quo of being an object with (full) paths as keys.

## Detail

At the moment, the manifest is formed of three components: the url_base (a string), the items (an object of type to object pairs, then an object giving each path and the items contained therein), and the path_hash (storing all the paths—again—and the type & file hash).

The proposal here is two-fold:

Firstly, replace the items objects with a series of nested objects forming a trie of path segments. This makes it much quicker to iterate through only specific directories, which is often enough done in a test-debug cycle.

Secondly, we get rid of path_hash: we migrate the hashes into the item object, thereby getting rid of the duplication of all the paths. This almost halves the size of the JSON created, which when the small-update case was previously dominated by parsing/serializing the JSON is significant.

Additionally, we stop storing the URL if it is identical to the path; this again provides size savings but is not a critical part of this proposal.

## Risks

As with any manifest format change, there is a risk that some consumers aren't paying attention to the version field and will fail loudly to cope with the new format.
