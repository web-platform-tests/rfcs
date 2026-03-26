# RFC 226: Exclusions in WEB_FEATURES.yml files

## Summary

Extend the schema of the `WEB_FEATURES.yml` files to support excluding one set
of tests from another.

## Background

[Contributors classified a large number of web-features over the final months
of
2025](https://github.com/search?q=repo%3Aweb-platform-tests%2Fwpt++is%3Apr+author%3Achrisc+author%3Ajugglinmike+author%3Astalgiag+author%3Ahoward-e++author%3Agnarf+created%3A%3E2025-09-01+&type=pullrequests),
and their experience surfaced many opportunities for improving the
maintainability of WPT's `WEB_FEATURES.yml` files[^1]. One of them is a
shorthand for excluding one set of tests from another set.

The problem starts with the application of filename-matching patterns. On its
face, this is a helpful feature. It helps maintainers avoid some of the
brittleness of the out-of-band classification scheme. However, in many cases,
there are subsets of tests which are better-classified to distinct
web-features. It's currently possible to exclude tests by filename by prefixing
a pattern with an exclamation mark (`!`), similar to [the convention in Git's
`.gitignore` files](https://git-scm.com/docs/gitignore). In the example below
(inspired by [the current form of
`html/webappapis/user-prompts/WEB_FEATURES.yml`](https://github.com/web-platform-tests/wpt/blob/e08956ee1ae70503cfbbc160716081768c9630cd/html/webappapis/user-prompts/WEB_FEATURES.yml)),
the entry for the `alerts` web-feature includes all files except those
belonging to the `print` web-feature:

```yaml
features:
- name: alerts
  files:
  - "*"
  - "!print-*"
- name: print
  files:
  - print-*
```

Maintaining this separation requires duplicating membership information in two
places. This duplication is challenging to maintain, and it obscures the
semantic relationship between the sets. (For larger excluded sets, the
challenge is more pronounced--the example used in this document was selected
for brevity, but there exist many files with far more duplication, e.g.
[`css/CSS2/generated-content/WEB_FEATURES.yml`](https://github.com/web-platform-tests/wpt/blob/e08956ee1ae70503cfbbc160716081768c9630cd/css/CSS2/generated-content/WEB_FEATURES.yml).)

Although in-line comments can alert contributors to the intention and implore
them to maintain it, a structural fix would alleviate everyone of this chore.

## Details

We propose allowing any entry for one web-feature to reference another from the
`files` list by prefixing the web-feature ID with the number sign (`#`). The
matched files can be excluded from the referencing set by further prefixing
with the exclamation point (`!`), just as with filename patterns. Filename
patterns should be distinguished with a leading sequence of `./`.

The example above, altered according to this proposed change, would appear as
follows:

```yaml
features:
- name: alerts
  files:
  - ./*
  - "!#print"
- name: print
  files:
  - ./print-*
```

## Alternatives Considered

### Changing the name of the `files` key

The patterns will no longer directly describe files, so "files" may not be the
more accurate name. Names like `patterns` or `tests` may be more appropriate.

```yaml
features:
- name: alerts
  patterns:
  - ./*
  - "!#print"
- name: print
  patterns:
  - ./print-*
```

However, file-matching continues to be the patterns' purpose. This might be
worth re-visiting when/if we decide to allow for matching tests as distinct
from files (e.g. matching specific expansions of [`.any.js`
tests](https://web-platform-tests.org/writing-tests/testharness.html#tests-for-other-or-multiple-globals-any-js)).

### Exclusion as the default (and only) behavior

Currently, we only have need to exclude sets. The `!` prefix, borrowed from
existing file-pattern syntax, insinuates non-existent "inclusion" use-case.
Omitting it would avoid any such insinuation and reduce the number of
characters necessary to support the desired use-case.

```yaml
features:
- name: alerts
  files:
  - ./*
  - "#print"
- name: print
  files:
  - ./print-*
```

That said, it seems somewhat confusing for the semantics of the two types of
patterns to vary so widely. (This could be mitigated by placing exclusions
under a separate key since. A name like "exclusions" would make it much easier
to recognize the "exclude by default" semantic of these patterns. However,
introducing a separate key by any name has its own drawbacks--see the next
section.) There's also reason to believe that future extensions will make use
of feature *inclusion* (namely: the ability to explicitly express intentional
overlaps).

### Expressing inclusions under a dedicated key

Introducing a new key like `features` (and placing all feature exclusion rules
there) would negate the need for "pattern type" prefixes altogether and obviate
all the design considerations which follow this one. Since we expect most
patterns to describe files directly, any prefix for those entries could be
considered noisy. (They will also dramatically increase the size of the patch
which enacts this RFC.)

For example:

```yaml
features:
- name: alerts
  files:
  - "*"
  features:
  - "!print"
- name: print
  files:
  - print-*
```

And for completeness, here's how that could look with the "exclude by default"
semantic contemplated above:

```yaml
features:
- name: alerts
  files:
  - "*"
  exclude:
  - print
- name: print
  files:
  - print-*
```

However, splitting patterns across two keys, whatever their name, has
concerning implications for pattern precedence. The sequence for applying two
distinct sets of patterns isn't immediately obvious (especially considering
that this feature may one day be used to express intentional overlaps between
sets). Whatever design we might select, the need for a dedicated design effort
suggests that the behavior may not be intuitive for contributors.

A more complex schema could have it both ways: a "list of maps of lists" would
reduce the repetition of a "files" designation while preserving the author's
control over the order of operations. For example, the rules expressed in the
following sketch have a much more clear sequence:

```yaml
features:
- name: alerts
  patterns:
  - files:
    - "*"
  - features:
    - "!print"
- name: print
  patterns:
  - files:
    - print-*
```

While clear precedence is important (especially for future extensions), the
complexity of this structure seems preferable to the noise of pattern prefixes.

### More explicit pattern prefixes

Keyword prefixes like `files:` and `feature:` would be more self-documenting
(and would likely justify the key-renaming change explored above):

```yaml
features:
- name: alerts
  patterns:
  - files:*
  - !feature:print
- name: print
  patterns:
  - files:print-*
```

The shorter prefixes (`./` and `#`) are inspired by conventions that many
developers will recognize (filesystem paths and HTML IDs, respectively), so
they strike a balance between verbosity and understandability.

### Omitting the `files:` prefix

The prefix is not technically necessary because tooling could assume entries
are filename patterns if they lack an explicit "feature" prefix (whatever that
prefix may be).

```yaml
features:
- name: alerts
  files:
  - "*"
  - "!#print"
- name: print
  files:
  - print-*
```

This inconsistency in the two pattern types may cause confusion for readers.
For what it's worth: the two-character `./` prefix obviates the need to apply
quotes to patterns which begin with `*`, so applying this change to those
amounts to simply swapping characters[^2].

### Omitting *both* prefixes

*Neither* prefix may be necessary to disambiguate the two types of patterns.
Today (and for the foreseeable future), all test filename patterns include
either an asterisk character (`*`) or a period character (`.`), and no
web-feature ID includes those characters. If we could secure an explicit
commitment from the WebDX community group about the makeup of web-feature IDs,
then we could lean on these details to infer pattern type based on their
content alone.

```yaml
features:
- name: alerts
  files:
  - "*"
  - "!print"
- name: print
  files:
  - ./print-*
```

While this disambiguation heuristic would be trivial to implement in software,
it's not particularly discoverable (and even after folks learn about it, it
feels cognitively burdensome to apply).

## Risks

### Schema modifications may interfere with external code

Any change to the schema of the `WEB_FEATURES.yml` file could interfere with
code beyond WPT that relies on those files.

However, these files are not generally promoted for external use. There exists
a far more ergonomic method for interested parties to access the relevant
information: the web-feature manifests. These manifests are:

- easier to parse (they are formatted in JSON rather than YAML)
- easier to interpret (they use explicit file paths rather than patterns and
  exclusions)
- easier to access (they are published using the "GitHub Releases" service,
  allowing consumers to download them directly rather than cloning the WPT
  repository)

### More expressive power may cause unintended classification of future contributions

Changes to the way tests are classified may appear to introduce new risks when
applying the patterns. Namely, when contributors introduce new tests, those
tests can be silently matched to the wrong web-feature.

While this is a risk, it is present in the current implementation of
`WEB_FEATURES.yml` files (specifically through the pattern-matching, or
"globbing", syntax). This RFC only simplifies how file-matching are expressed;
it does not introduce a fundamentally different mechanism for matching files.

### Complexity may discourage contribution

The more complex a custom schema becomes, the more effort contributors have to
spend in order to participate.

The fact that this functionality is purely additive reduces the load for
newcomers. Even without understanding the exclusion syntax, they can express
the relationships--albeit in a more literal and less maintainable way. It can
be the prerogative of reviewers (or future contributors) to revise such patches
with the new syntax, where applicable.

We also intend to supplement this change with further linting rules, such as
verifying the existence of web-feature identifiers. This will provide immediate
and contextual feedback to contributors who may be experimenting with the
syntax for the first time.

[^1]: `WEB_FEATURES.yml` files were introduced via [RFC
      163](https://github.com/web-platform-tests/rfcs/blob/main/rfcs/web_features.md).
[^2]: There are currently 212 such patterns

          $ find . -name WEB_FEATURES.yml -print0 | xargs -0 grep "\- [\"']\*" | wc -l
          212
