# RFC 198: Move the container image from Docker Hub to GitHub Registry

## Summary

This RFC proposes moving the container image from Docker Hub to GitHub Registry.

## Details

This RFC addresses the issue
[#28903](https://github.com/web-platform-tests/wpt/issues/28903). Docker Hub
organization allows for a limited number of members and only a few members can
publish a new Docker image that sometimes results in the Docker image that is
used in taskcluster being outdated as documented in [this PR](https://github.com/web-platform-tests/wpt/pull/44576#issue-2133724788).

The following are the benefits of using the GitHub registry for hosting:

- It is [hosted on GitHub](https://github.com/web-platform-tests/wpt/packages) together with the wpt repository.
- Publishing is possible by any project member who has the write permissions to the
  repository.
- Publishing is possible [via GitHub Actions](https://github.com/web-platform-tests/wpt/blob/master/.github/workflows/docker.yml):

    - The workflow can be manually invoked,
    - or it can be further automated to deploy any changes that land.

- We can take the opportunity to change the versioning scheme to use integer numbers.

The following PR implements changes to use the image from GitHub registry
instead of the Docker Hub one
https://github.com/web-platform-tests/wpt/pull/46279 including modifications to
tools/docker/frontend.py.

This RFC also proposes to change the version numbers to be integers starting
with `1`.

## Risks

Users of the current image hosted at Docker Hub would need to update the image
name from `webplatformtests/wpt:0.57` to `ghcr.io/web-platform-tests/wpt:1` (or
whatever is the latest version). The GitHub Registry image is publicly available
like the Docker Hub one but there could be some friction with the update
depending on the user's setup.