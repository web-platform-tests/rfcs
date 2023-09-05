# RFC #163: Add web_feature metadata file to web-platform-tests

Author: @jcscottiii

# Introduction

The web-platform-tests (WPT) project is a valuable resource for testing web
platform features and contains valuable metadata such as feature spec links.
Recently, the [WebDX Community Group](https://www.w3.org/community/webdx/)
began creating the [feature-set](https://github.com/web-platform-dx/feature-set)
repository. That repository serves as a basic shared catalog of feature
definitions of the web platform. Feature-set itself doesn't intend to produce
data but rather link to existing data that will inform audiences which features
are part of [Baseline](https://web.dev/baseline/). However, there's a great
opportunity to connect the WPT ecosystem to the feature-set catalog. By doing
this, it would enable users of wpt.fyi to filter by feature-set grouping, which
is similar to
[the ability to filter by spec links](https://github.com/web-platform-tests/wpt.fyi/issues/1489).
The RFC proposes the addition of a WEB_FEATURE.yml metadata file which would
enable the linkage between WPT and feature-set.

# Proposed change

The proposed change includes:

1. Introduce a new metadata file, WEB_FEATURE.yml
2. Create a script to generate a manifest
3. Adjust the wpt-pr-bot to handle reviews of the changes

Below are the details for each step.

## Step 1. Introduce a new metadata file, WEB_FEATURE.yml

Originally, there was a RFC that proposed adding these details to the existing
META.yml. During the August 01, 2023 Monthly WPT
[Meeting](https://github.com/web-platform-tests/wpt-notes/blob/master/minutes/2023-08-01.md#rfc-157---add-feature-meta-tag-to-web-platform-tests-meta-data),
it was discussed that it should be a separate file. This section describes the structure of the file.

This file is expected to be in the same places developers would expect a META.yml.

### File examples

Typical example:

```
web_feature: subgrid
```

An example using all of the fields:

```
apply_mode: FORCE_RECURSIVE
web_feature: feature1
overrides:
- file_name: name.txt
  web_feature: feature2
  override_mode: REPLACE
```

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
                "web_feature": {
                    "type": "string",
                    "description": "The web feature key"
                },
                "apply_mode": {
                    "type": "string",
                    "anyOf": [
                    	{
                            "const": "DEFAULT",
                            "description": "Applies recursively until the presence of another WEB_FEATURE.yml in a subdirectory."
                        },
                    	{
                            "const": "FORCE_RECURSIVE",
                            "description": "Applies recursively throughout the all sub-directories. Useful if web_feature hierarchy exactly matches wpt directory structure."
                        }
                    ],
                    "default": "DEFAULT"
                },
                "overrides": {
                    "type": "array",
                    "items": {
                        "$ref": "#/definitions/Override"
                    }
                }
            },
            "required": [
                "web_feature"
            ],
            "title": "Main"
        },
        "Override": {
            "type": "object",
            "additionalProperties": false,
            "properties": {
                "file_name": {
                    "type": "string",
                    "description": " The file name of the test in the same directory as this file."
                },
                "web_feature": {
                    "type": "string",
                    "description": "The web feature key"
                },
                "override_mode": {
                    "type": "string",
                    "anyOf": [
                    	{
                            "const": "APPEND",
                            "description": "Append to the current web_feature key(s) for the given file"
                        },
                    	{
                            "const": "REPLACE",
                            "description": "Replace the current web_feature key(s) for the given file"
                        },
                    ],
                    "default": "APPEND"
                },
            },
            "required": [
                "web_feature",
                "file_name"
            ],
            "title": "Override"
        }
    }
}
```

The next section describes the manifest generation process section.

## Step 2. Create a script to generate a manifest

A command will be created in the `tools/manifest` folder to generate the
WEB_FEATURE_MANIFEST.json file. While it is in that folder, its logic will
be independent of the existing manifest logic. (As discussed in the August 01,
2023 Monthly WPT
[Meeting](https://github.com/web-platform-tests/wpt-notes/blob/master/minutes/2023-08-01.md#rfc-157---add-feature-meta-tag-to-web-platform-tests-meta-data),
it should not use the same logic.) The only thing it will it
do is generate data in the same structure.

By using the same format, wpt.fyi can use it to allow filtering by web feature.

The rules of the generation are described in the file schema above by the
directory level `apply_mode` flag and the per file `override_mode`.

## Step 3. Adjust the wpt-pr-bot to handle reviews of the changes

### Current State of wpt-pr-bot

Currently, the wpt-pr-bot builds a list of PR reviewers by:
1. Retrieving the paths for all of the files
2. Finding the nearest META.yml file for each path
3. Add the list of suggested_reviewers from a given META.yml to a set of
   reviewers

### Proposed changes

Have the wpt-pr-bot filter the WEB_FEATURE.yml file changes to only request
reviews from feature-set contributors.

This will reduce the amount of unneeded reviews from non feature-set contributors.

---

# Risks

As a result of putting all the information in WEB_FEATURE.yml, the overrides
which are linked to the file can go out of sync as test files are moved,
renamed, or deleted.

## Short Term Mitigation

While desired, there is no anticipated percentage of tests that will use the
override functionality. As a result, an outside process could alert when there
is some issue with the data. The web feature team will be responsible for that.

If it becomes a problem to manage, one of the following future mitigation options
in the next section can solve it.

## Future Mitigation

This RFC does not cover the a long term mitigation plan for the risk. However,
a possible mitigation could be one of the following:
- A new GitHub Action that can prevent a PR from being merged if a user
  modifies a overridden test without modifying the WEB_FEATURE.yml, or
- Expand on the wpt-pr-bot changes to require a WEB_FEATURE reviewer if it
  detects a overridden test has changed (even if WEB_FEATURE.yml has not).

# Other Notes

## Changes for WPT Contributors

The metadata tag is not mandatory. WPT contributions will not be blocked if
contributors do not add the metadata tag to the WEB_FEATURE.yml files or test files.

## Populating and maintaining the metadata files

As the [feature-set](https://github.com/web-platform-dx/feature-set) repository
is populated with feature-set definitions, feature-set contributors will begin
populating the metadata in WPT.

In regards to maintaining the metadata files, feature-set contributors will be
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
