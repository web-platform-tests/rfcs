# RFC #80: Rename `master` branch to `main` in all repositories

## Summary

Rename `master` branch to `main` in all repositories in the [web-platform-tests org](https://github.com/web-platform-tests).

## Details

### wpt.fyi

[https://github.com/web-platform-tests/wpt.fyi](https://github.com/web-platform-tests/wpt.fyi)

#### Run labelling

**Example**: [https://wpt.fyi/api/runs?label=master](https://wpt.fyi/api/runs?label=master)

Today we use the term 'master' to label runs that come from either pushes to the master branch in
WPT (for Chrome Dev and Firefox Nightly) or from the various epoch/\* branches for other browsers
and channels. These runs should be complete runs of the full test suite (as opposed to partial runs
done for pull-requests), so are considered the most useful for displaying on wpt.fyi.

There are two considerations to how this label is used:

1.  Internally to the service (the variable/method names 'shared.MasterLabel' or 'IsMasterBranch',
    values stored in the datastore, etc) - these could be renamed without too much difficulty.

    - For example, one could consider renaming the concept to 'complete' runs, such that we had
      'shared.CompleteRunLabel'.

    - Method names like IsMasterBranch are harder to change until WPT changes its branch naming
      convention.

2.  Externally accessible information, e.g. as used in the APIs. We allow querying runs based on
    their label, and both the [wpt.fyi](http://wpt.fyi) frontend and other callers may
    rely on \`label=master\` today.

    - This would be a slow process of introducing an alternative label (again say 'complete'),
      labelling runs as both in the datastore (data fixups can be done with [existing
      utilities](https://github.com/web-platform-tests/data-migration)), and then slowly
      switching over consumers to the new label name.

There are approximately 200 references to the string 'master' in the wpt.fyi codebase:

- 106 in api/

- 60 in webapp/

- 28 in shared/

- 4 in util/

- 4 in webdriver/

#### Primary branch name

~~Today wpt.fyi has 'master' as its primary branch name in GitHub. Changing this to e.g. 'main'
would require at least:~~

- ~~Creating the new branch ('git branch -m master main; git push -u origin main'),~~

- ~~Changing the primary branch in GitHub,~~

- ~~Editing all open pull requests (currently only 5) to have 'main' as their target,~~

- ~~Switching the branch protection rules to apply to 'main', and~~

- ~~Updating the CI workflows (e.g. \`.github/workflows/ci.yml\`) to run on the new branch.~~

- ~~Deleting the master branch~~

~~There is also a reference to 'master' in the gcloud deployment steps in the Makefile.~~

~~I am not aware of any downstream consumers of the branch name for wpt.fyi, so we could likely do
the introduction of 'main' as a mostly-atomic move with no prerequisites.~~

### WPT

[https://github.com/web-platform-tests/wpt](https://github.com/web-platform-tests/wpt)

#### Primary branch name

Today wpt.fyi has 'master' as its primary branch name in GitHub. Changing this to e.g. 'main' would
have similar steps [as for wpt.fyi](#wpt.fyi) (e.g. changing the primary GitHub
branch, branch protection, etc), but with more complexity due to having more continuous integration
(CI) systems and many downstream consumers.

##### Outstanding Pull Requests

WPT currently has 754 open pull requests, almost all targeting the 'master' branch. These would all
need to be updated to target the 'main' branch instead.

This can be done manually by maintainers (by editing the pull request via the GitHub API), but it
appears that we could also use the [GitHub REST
API](https://developer.github.com/v3/pulls/#update-a-pull-request) to do this via PATCH
requests to open pull requests (changing the 'base' branch to e.g. 'main').

#### Continuous Integration systems

##### Taskcluster

One of our two CI systems for running the test suite on browsers. Taskcluster has a few
dependencies on the name 'master', but all could be done ahead of a switchover.

- .taskcluster.yml would need 'main' added as a trigger branch

- tools/ci/tc/tasks/test.yml would need 'main' added as a trigger branch

- tools/ci/tc/download.py uses 'master' as the default branch for its '--ref' argument.

  - This is the tc-download wpt command, but it is unclear if it is used in any automated setting -
    perhaps this could be changed after the fact.

- tools/docker/start.sh uses 'master' as a default ref, but it appears that the only caller
  (tools/ci/tc/decision.py) passes an explicit ref anyway.

  - This could possibly just be removed.

##### Azure Pipelines

Our other CI system for running the test suite on browsers. Azure Pipelines does not run on
'master' today, so I believe it requires no changes in the case of a branch name switch.

##### Pull-request previews (wptpr.live integration)

This is a pair of GitHub Actions that support previewing pull requests on wptpr.live. For the
actions themselves (see [Downstream Consumers of WPT branch
names](#downstream-consumers-of-wpt-branch-names) for the wptpr.live side), there are two
places that reference 'master':

- .github/workflows/detect_pull_request_preview.yml deliberately checks out the 'master' branch
  specifically.

  - This would have to change to 'main', and would have to happen as part of the 'atomic'
    switchover.

- tools/ci/tests/test_pr_preview.py uses it in the testing logic as part of a set; this can be
  changed at any point (the test creates a temporary git repository via git-init, so that would
  have to change to not create a 'master' branch).

##### Manifest generation

This is a GitHub Action that publishes WPT MANIFEST.json files as [GitHub
releases](https://github.com/web-platform-tests/wpt/releases). There are two places that
reference 'master':

- .github/workflows/manifest.yml triggers on pushes to master; this could be changed ahead of time
  to also trigger on pushes to e.g. 'main'

- tools/ci/manifest_build.py checks for 'master' to determine if this is a dry-run or not; another
  branch name could be added to this check ahead of time.

##### Epochs

No obvious dependency. (Uses the default branch when checking out the code.)

#### Documentation

WPT has a sizeable amount of documentation (in docs/), which is generated by a GitHub action.
Changes to both the generation process and the documentation itself would be necessary:

**docs/ generation**:

- .github/workflows/documentation.yml triggers on push to master; this could be changed ahead of
  time to also trigger on pushes to e.g. 'main'

- tools/ci/website_build.sh checks for 'master' to only publish when run via the GitHub Action;
  this could be changed ahead of time to also publish for e.g. 'main'.

**docs/ documentation**:

- Three files have substantial content that mention 'master':

  - docs/reviewing-tests/git.md

  - docs/reviewing-tests/reverting.md

  - docs/writing-tests/github-intro.md

- Plus some minor links/etc in other documents.

**README.md**

- There are a few minor notes about 'master' here.

#### WPT tools/ codebase

There are a few places in the WPT infrastructure code that interact with 'master'.

##### wpt branch-point

- tools/wpt/testfiles.py checks for 'master' in 'branch_point()', would need changed as part of the
  migration to use e.g. 'main'.

- **Note**: it is unclear to me who uses this tool?

##### wptrunner

- tools/wptrunner/wptrunner.default.ini references 'master', but I'm unsure of the purpose of this
  file.

#### Uses of 'master' in test suites

Some minor documentation changes (e.g. in css/README.md) would be required, but would be
non-blocking.

#### Downstream Consumers of WPT branch names

This is one of the trickiest areas of changing the default branch name in WPT; there are multiple
consumers of WPT that rely on the branch name today.

The ones I am aware of are:

- wpt.fyi:

  - Would need to be updated to accept either 'master' or (e.g.) 'main' ahead of a switch of branch
    name. See [wpt.fyi](#wpt.fyi) above.

- wpt.live:

  - src/supervisord.conf pulls from the 'master' branch specifically, this would need updated.

  - Similarly, 'master' is passed to fetch-wpt.py as the branch to fetch from.

  - These would be difficult to update ahead of time; they could possibly be removed entirely and
    rely on the default branch.

- w3c-test.org:

  - This may also pull directly from master like wpt.live, unknown.

- There is also the web-test runner, not sure if that has any dependencies on WPT branch names.

- Various sync bots (might have “master” hard coded)

  - autofoolip

  - wpt-pr-bot

- Downstream mirrors

  - Chromium mirror:
    [https://chromium.googlesource.com/external/github.com/web-platform-tests/wpt/](https://chromium.googlesource.com/external/github.com/web-platform-tests/wpt/)
    (used by wpt-importer to avoid GitHub network issues)

## Risk

Calamity
