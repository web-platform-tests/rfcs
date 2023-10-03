# RFC #163: Add web_feature metadata file to web-platform-tests

Author: @jcscottiii

# Introduction

The web-platform-tests (WPT) project is a valuable resource for testing web
platform features and contains valuable metadata such as feature spec links.
Recently, the [WebDX Community Group](https://www.w3.org/community/webdx/)
began creating the [web-features](https://github.com/web-platform-dx/web-features)
repository. That repository serves as a basic shared catalog of feature
definitions of the web platform. The repository itself doesn't intend to produce
data but rather link to existing data that will inform audiences which features
are part of [Baseline](https://web.dev/baseline/). However, there's a great
opportunity to connect the WPT ecosystem to the web-features catalog. By doing
this, it would enable users of wpt.fyi to filter by web-features grouping, which
is similar to
[the ability to filter by spec links](https://github.com/web-platform-tests/wpt.fyi/issues/1489).
The RFC proposes the addition of a WEB_FEATURE.yml metadata file which would
enable the linkage between WPT and web-features.

# Proposed change

The proposed change includes:

1. Introduce a new metadata file, WEB_FEATURE.yml
2. Add linting functionality
3. Create a script to generate a manifest
4. Adjust the wpt-pr-bot to handle reviews of the changes

Below are the details for each step.

## Step 1. Introduce a new metadata file, WEB_FEATURE.yml

Originally, there was a RFC that proposed adding these details to the existing
META.yml. During the August 01, 2023 Monthly WPT
[Meeting](https://github.com/web-platform-tests/wpt-notes/blob/master/minutes/2023-08-01.md#rfc-157---add-feature-meta-tag-to-web-platform-tests-meta-data),
it was discussed that it should be a separate file. This section describes the structure of the file.

This file is expected to be in the same places developers would expect a META.yml.

### File examples

#### Example 1: Default

```yaml
features:
   - name: feature1
     files: "**"
```

#### Example 2: A directory which has multiple features:

```yaml
features:
- name: test-2
  files:
  - test-2-01.html
  - test-2-02.html
- name: test-1
  files:
  - test-1-01.html
```

#### Example 3: Ignore parent features

```yaml
apply_mode: IGNORE_PARENT
features:
- name: reset-test-1
  files: "**"
```

#### Example 4: Ignore parent features and listing out explicit files

```yaml
apply_mode: IGNORE_PARENT
features:
- name: reset-test-2
  files:
  - reset-test-2-01.html
- name: reset-test-1
  files:
  - reset-test-2-01.html
```

### Evidence for the above examples:
- Example 2 is necessary for directories such as
  https://wpt.fyi/results/css/selectors which has tests for multiple features.
- Examples 3 and 4 are necessary to support marking up all of
  https://wpt.fyi/results/css/css-grid *except* the masonry/ and subgrid/
  subdirectories as being part of the grid feature.

### Schema

```json
{
    "$schema": "http://json-schema.org/draft-06/schema#",
    "$ref": "#/definitions/Main",
    "definitions": {
        "Main": {
            "type": "object",
            "additionalProperties": false,
            "properties": {
                "features": {
                    "type": "array",
                    "description": "List of features",
                    "items": {
                        "$ref": "#/definitions/FeatureEntry"
                    }
                },
                "apply_mode": {
                    "type": "string",
                    "anyOf": [
                    	{
                            "const": "IGNORE_PARENT",
                            "description": "Ignores features from previous parent directories."
                        }
                    ]
                }
            },
            "required": [
                "features"
            ],
            "title": "Main"
        },
        "FeatureEntry": {
            "type": "object",
            "additionalProperties": false,
            "properties": {
                "name": {
                    "type": "string",
                    "description": "The web feature key"
                },
                "files": {
                    "oneOf": [
                        {
                            "const": "**",
                            "description": "All files recursively",
                            "title": "Files"
                        },
                        {
                            "type": "array",
                            "items": {
                                "type": "string",
                                "description": "A specific file within the current directory"
                            }
                        }

                    ]
                }
            },
            "required": [
                "name",
                "files"
            ],
            "title": "FeatureEntry"
        }
    }
}
```

## Step 2. Add linting functionality

There exists a potential risk of putting all the information in WEB_FEATURE.yml:
the overrides which are linked to the file can go out of sync as test files are moved,
renamed, or deleted.

As a result, this RFC also proposes a lint command to detect this. The code for
this will reside in `tools/lint/lint.py`.

## Step 3. Create a script to generate a manifest

A command will be created in the `tools/manifest` folder to generate the
WEB_FEATURE_MANIFEST.json file. While it is in that folder, its logic will
be independent of the existing manifest logic. (As discussed in the August 01,
2023 Monthly WPT
[Meeting](https://github.com/web-platform-tests/wpt-notes/blob/master/minutes/2023-08-01.md#rfc-157---add-feature-meta-tag-to-web-platform-tests-meta-data),
it should not use the same logic.) The only thing it will it
do is generate data in the same structure.

By using the same format, wpt.fyi can use it to allow filtering by web feature.

The rules of the generation are described in the file schema above by the
directory level `apply_mode` flag.

## Step 4. Adjust the wpt-pr-bot to handle reviews of the changes

### Current State of wpt-pr-bot

Currently, the wpt-pr-bot builds a list of PR reviewers by:
1. Retrieving the paths for all of the files
2. Finding the nearest META.yml file for each path
3. Add the list of suggested_reviewers from a given META.yml to a set of
   reviewers

### Proposed changes

Have the wpt-pr-bot filter the WEB_FEATURE.yml file changes to only request
reviews from web-features contributors.

This will reduce the amount of unneeded reviews from non web-features contributors.

---

# Risks

A set of new linting cases (mentioned in Step 2.) could confuse existing
developers.

## Mitigation

Ways to mitigate this include:
- Informative logging
- Additional documentation to describe why this linting exists.

# Other Notes

## Changes for WPT Contributors

The metadata tag is not mandatory. WPT contributions will not be blocked if
contributors do not add the metadata tag to the WEB_FEATURE.yml files or test files.

## Populating and maintaining the metadata files

As the [web-features](https://github.com/web-platform-dx/web-features) repository
is populated with web-features definitions, web-features contributors will begin
populating the metadata in WPT.

In regards to maintaining the metadata files, web-features contributors will be
responsible for the mitigation options
described above. 
# Roll back of this RFC

The following steps will allow the community to roll back this RFC in the event it is deemed unnecessary:

1. Remove all the new metadata files
    - ```sh
        find . -name WEB_FEATURE.yml -type f -delete
        ```
2. Remove the new web_feature.py and reference in commands.json
    - This will likely happen by reverting the PRs in the tools/manifest folder.
3. Remove the lint code in tools/lint/lint.py.
    - This will likely happen by reverting the PRs in the tools/lint/lint.py file.
