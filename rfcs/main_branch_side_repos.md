# RFC #225: Rename `master` branch to `main` in non-test repositories

## Summary

Rename `master` branch to `main` in 
[web-platform-tests/data-migration](https://github.com/web-platform-tests/data-migration),
[web-platform-tests/wpt.live](https://github.com/web-platform-tests/wpt.live),
[web-platform-tests/rfcs](https://github.com/web-platform-tests/rfcs),
[web-platform-tests/editor](https://github.com/web-platform-tests/editor),
[web-platform-tests/code-of-conduct-moderations](https://github.com/web-platform-tests/code-of-conduct-moderations), and
[web-platform-tests/wpt-notes](https://github.com/web-platform-tests/wpt-notes).


## Details

Find non-archived repositories whose default branch is not main with:

```sh
gh api /orgs/web-platform-tests/repos | jq -r '.[] | select(.default_branch != "main" and (.archived | not)) | .html_url + ": " + .default_branch'
```

Yields:

```
https://github.com/web-platform-tests/wpt: master
https://github.com/web-platform-tests/data-migration: master
https://github.com/web-platform-tests/wpt.live: master
https://github.com/web-platform-tests/rfcs: master
https://github.com/web-platform-tests/wpt-metadata: master
https://github.com/web-platform-tests/editor: master
https://github.com/web-platform-tests/code-of-conduct-moderations: master
https://github.com/web-platform-tests/wpt-notes: master
```

This RFC proposes to rename the default branches of everything *except* `wpt` and `wpt-metadata` to `main`.

Specifically:

### `data-migration`

There are no references to the `master` branch of this repo.
No further work is expected.

### `wpt.live`

This contains [a few references](https://github.com/search?q=repo%3Aweb-platform-tests%2Fwpt.live+master&type=code) to `master`.
Two of these appear to refer to the repo itself.
Both can be fixed to be generic.

### `rfcs`

There are no code references to the `master` branch of this repo.
No further work is expected.

### `editor`

There are no code references to the `master` branch of this repo.
No further work is expected.

### `code-of-conduct-moderations`

This contains no references to any `master` branch.
No further work is expected.

### `wpt-nodes`

There are no code references to the `master` branch of this repo.
No further work is expected.


## Risk

Calamity
