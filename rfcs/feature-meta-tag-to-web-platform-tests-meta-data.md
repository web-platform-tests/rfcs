# RFC #157: Add feature meta tag to web-platform-tests meta data

Author: @jcscottiii

# Introduction

The web-platform-tests (WPT) project is a valuable resource for testing web platform features and contains valuable metadata such as feature specs. Recently, the [WebDX Community Group](https://www.w3.org/community/webdx/) began creating the [feature-set](https://github.com/web-platform-dx/feature-set) repository. That repository serves as a basic shared catalog of feature definitions of the web platform. Feature-set itself doesn't intend to produce data but rather link to existing data that will inform audiences which features are part of [Baseline](https://web.dev/baseline/). However, there's a great opportunity to connect the WPT ecosystem to the feature-set catalog. By doing this, it would enable users of wpt.fyi to filter by feature-set grouping, which is similar to [the ability to filter by spec](https://github.com/web-platform-tests/wpt.fyi/issues/1489). The RFC proposes the addition of a "feature" meta tag which would enable the linkage between WPT and feature-set.

# Proposed change

The proposed change is to add a "feature" meta tag in two potential locations:

1. As a high level key in a META.yml, or
1. As a `<meta>` annotation within a specific test.

The two following sections discuss each respective location in more detail.

## High Level "feature" Key inside META.yml

Within the META.yml, the "feature" meta tag would be a high level YAML key. The key value is a string that maps to the one of the feature definitions in the feature-group-definitions folder of feature-set. This will cover most cases.

### Example

Here's an example of the changes needed to create the mapping in WPT's [css/css-grid/META.yml](https://github.com/web-platform-tests/wpt/blob/master/css/css-grid/META.yml) to feature-set's [grid.yml](https://github.com/web-platform-dx/feature-set/blob/main/feature-group-definitions/grid.yml)

```diff
spec: https://drafts.csswg.org/css-grid/
+ feature: grid
suggested_reviewers:
  - mrego
  - plinss
  - tabatkins
  - fantasai
  - javifernandez
  - rachelandrew
  - svillar

```

### Extra case to consider and implications: Nested META.yml files

There are a few instances where there is a directory that contains a META.yml file and that same directory has a subdirectory that also has a META.yml (Example: [css/css-text/META.yml](https://github.com/web-platform-tests/wpt/blob/master/css/css-text/META.yml) and [css/css-text/i18n/META.yml](https://github.com/web-platform-tests/wpt/blob/master/css/css-text/i18n/META.yml))

Building on the above grid example, there could be a css/css-grid/subgrid/META.yml. That META.yml would have a feature key that points to the feature-set subgrid.yml file.

```yaml
feature: subgrid
```

**How to handle this:** When it comes to nested META.yml files, only the closest META.yml file with a feature key will be considered.  
**Expected behavior:** When it comes to filtering by feature and the supplied feature is "grid", it will not include the "subgrid" tests.

## Test specific feature annotations

Similar to the use of `<link rel="help">` to signify additional specs on a per-test level, there will be a similar implementation for features. This is to cover any edge cases where tests cover multiple features. Most implementations will use the way documented in the "High Level "feature" Key inside META.yml" section above.

### Example

Here's an example of annotating a specific test with a feature:

```diff
<!DOCTYPE html>
<title>Flex gaps</title>
<link rel="author" title="Author Name" href="mailto:test@chromium.org">
<link rel="help" href="https://drafts.csswg.org/css-flexbox/#main-alignment">
+ <meta name="feature" content="grid">
```

# Alternatives considered

One alternative considered was to add specific web platform tests to each feature-set definition instead. This would be similar to how BCD data is included in the definition ([Example](https://github.com/web-platform-dx/feature-set/blob/c9314c5126e74f4e1fecd56202774a845c66f9b4/feature-group-definitions/subgrid.yml#L9-L10)).  However, this would cause a developer experience where WPT authors would need to land changes in both WPT and feature-set. Given that test names in WPT change more frequently than feature group definitions, it could add unnecessary toil to maintain the connection.

# Implementation Details

There are two steps that need to happen:  
1. Annotate the META.yml files with the "feature" key that corresponds to the feature-set entry. This can be done gradually and not all at once.
2. Create a wpt script that generates a Feature manifest. Similar to the [SPEC_MANIFEST.json](https://github.com/web-platform-tests/wpt/pull/40655). For more details, check the diagram attached to the pull request for this RFC.

# Changes for WPT Contributors

The metadata tag is not mandatory. WPT contributions will not be blocked if contributors do not add the metadata tag to the META.yml files or test files.

# Populating and maintaining the metadata tags

As the [feature-set](https://github.com/web-platform-dx/feature-set) repository is populated with feature-set definitions, feature-set contributors will begin populating the metadata in WPT.

In the event a feature's key in feature-set changes, a feature-set contributor will update the tag in WPT.

In the event the 1) META.yml file with a `feature` key or 2) test file containing feature-set metadata is moved or deleted in the WPT repository, it is currently outside the scope of WPT to ensure data correctness for the feature-set metadata.

# Roll back of this RFC

The following steps will allow the community to roll back this RFC in the event it is deemed unnecessary:

1. Remove all annotations in the META.yml files
  - ```sh
    # Remove all the lines that start with feature
    find . -name META.yml -type f -print0 | xargs -0 sed -i '/^feature/d'
    # If the "feature" key was the only key in the file, remove the whole file.
    find . -name META.yml -type f -empty -delete
    ```
2. Remove all lines in the test files
  - ```sh
    # In case there are any spacing or ordering differences.
    find . -type f -print0 | xargs -0 sed -i '/^.*<meta.*name.*=.*"feature"/d'
    ```
