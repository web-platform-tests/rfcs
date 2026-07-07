# RFC 237: Exclusions in WEB_FEATURES.yml files

## Summary

Redesign the schema of the `WEB_FEATURES.yml` files to reduce repetition and
assist human readers.

## Background

[Contributors classified a large number of web-features over the final months
of
2025](https://github.com/search?q=repo%3Aweb-platform-tests%2Fwpt++is%3Apr+author%3Achrisc+author%3Ajugglinmike+author%3Astalgiag+author%3Ahoward-e++author%3Agnarf+created%3A2025-09-01..2026-05-01&type=pullrequests),
and their experience surfaced ways in which the design of WPT's
`WEB_FEATURES.yml` files[^1] is not optimized for their purpose.

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

In this design, multiple rules can match the same file, and when they do, the
files are implicitly assigned to multiple web-features. Cross-cutting tests are
certainly present in WPT, but it is far more common to express membership as
mutually-exclusive. By simplifying a rare use-case while frustrating a common
use-case, the current design is not optimally suited for its purpose.

## Details

We propose re-structuring the files to declare file pattern rules in a flat
list (where every entry individual entry also specifies the identifier of the
web-feature), and to apply a "first pattern wins" heuristic when generating the
file manifest.

The example above, altered according to this proposed change, would appear as
follows:

```yaml
rules:
- print-*: [print]
- "*": [alerts]
```

This design reduces the need for the exclusion syntax currently expressed with
an exclamation point (`!`) prefix. In cases where a given file should belong to
one web-feature but not another (as in the example above), exclusion can be
expressed through careful sequencing of rules.

This proposal retains the special `"**"` value originally proposed in [RFC
#153](https://github.com/web-platform-tests/rfcs/blob/main/rfcs/web_features.md).
That value continues to match all tests in the current directory and all tests
in any subdirectories. It is subject to the same sequence-based precedence as
the other patterns.

In the example above, to classify all tests in all sub-directories (along with
all tests in the current directory except for those prefixed with `print-`)
with the `alerts` web-feature, one would write:

```yaml
rules:
- print-*: [print]
- "**": [alerts]
```

In order to support the case of cross-cutting tests, it should be possible to
express membership in multiple web-features via a list of web-feature IDs.

In the example above, to include the file named `foo.html` in both the `print`
web-feature *and* the `alerts` web-feature, one would write:

```yaml
rules:
- foo.html: [print, alerts]
- print-*: [print]
- "**": [alerts]
```

Finally, careful rule ordering alone cannot address cases where files should be
excluded from *all* web-features. As one might infer from the schema's support
of collections of web-feature IDs, this proposal designates the empty list as
a signifier for files which should not belong to any particular web-feature.

In the example above, to exclude the file named `bar.html` from *any*
web-feature, one would write:

```yaml
rules:
- foo.html: [print, alerts]
- bar.html: []
- print-*: [print]
- "**": [alerts]
```

## Alternatives Considered

### A feature-based exclusion syntax

The problem of set exclusion could be addressed without changes to the
structure or semantics of the `WEB_FEATURES.yml` files via a new micro-syntax
for excluding entire sets.

In the example above, this might be expressed as follows:

```yaml
features:
- name: alerts
  files:
  - ./*
  - "!#print" # exclude all files belonging to the `print` web-feature
- name: print
  files:
  - ./print-*
```

However, this design increases the complexity of the schema. Any complexity
reduces the likelihood that these files will be organically maintained by
typical WPT contributors (since they are generally more focused on test
material than on metadata).

This design also does not address the mismatch between ergonomics and use-case
frequency. That is: it continues to favor the rare "overlap" case over the
common "exclude" case.

### Changing the semantics but not the schema

This proposal's strength comes from making file-matching rules mutually
exclusive by default. Because these rules have a deterministic order under the
existing schema, the main benefits could be enjoyed without any schema changes
whatsoever.

However, less structure is generally easier for human contributors to read and
understand. A simpler structure will tend to promote maintenance by those who
have less awareness of WPT infrastructure.

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
"globbing", syntax). This RFC only simplifies how file-matching is expressed;
it does not introduce a fundamentally different mechanism for matching files.

### Verbosity of naming web-feature identifiers on a per-file-pattern basis

This proposal's schema modifications involve declaring a web-feature ID for
every file-matching pattern. When web-features have many such patterns, the
repetition of their ID will tend to make the restructured `WEB_FEATURE.yml`
files more verbose than they would be using the original schema.

In practice, however, web-features typically have a small number of patterns.
At the time of this RFC, the mean number of file-matching patterns per
web-feature in a given directory[^2] is approximately 2.362 with a standard
deviation of approximately 5.7487. This reduces the benefit of optimizing for
the case of web-features with many associated file-matching patterns.

### Extendability

There is some expectation among maintainers that the rules in these files will
one day need to include additional structural metadata. The prior design
included dictionary values with well-defined keys, making it trivial to extend
the schema with support for new metadata. Because the design in this proposal
uses dictionary keys for data (i.e. file patterns) rather than structure, it is
not so readily extensible for metadata.

This design contemplates accommodating any future need for metadata through an
alternate and more verbose form of rule definition. For example, rules whose
could be expressed as a nested dictionaries in cases that need additional
metadata, and all other rules could go unchanged (continuing to use the more
concise syntax described by this proposal).

```yaml
rules:
- print-*: [print]
- "*":
    ids: [alerts]
    metadata_name: some metadata value
```

[^1]: `WEB_FEATURES.yml` files were introduced via [RFC
      163](https://github.com/web-platform-tests/rfcs/blob/main/rfcs/web_features.md).
[^2]: These statistics exclude `!`-prefixed entries because this proposal
      obviates such entries.
