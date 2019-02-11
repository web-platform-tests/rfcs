# RFC 9

## Summary

Use PRs rather than issues for the RFC process.

## Details

Each RFC should be a markdown-formatted document in the rfcs/ subdirectory. Creating the PR will signal the the RFC is ready for review and merging it will indicate that it's accepted. Conversely closing it will be used to indicate the RFC is rejected.
The RFC document should get a title that is its PR number (added after submission) and consists of a summary of the proposed changes, full details of the proposal and a discussion of pros and cons.

## Advantages

* PRs involve an actual document that can be revised after change and still has the version control expexted of a git repo. Although this is possible with comments, it's much less expected that a user would  have to examine the history of a comment to understand the discussion, and the tooling for exploring that history is weaker.
* Commiting the RFC produces a record of decisions that are made.
* Allowing people to clone the repo and use existing RFCs as a template helps ensure a consistent structure.
* Comments on specific spec can be made inline on that text.

## Disadvantages
* The overhead of creating a PR is slightly higher than for creating an issue (although both can be done fully in the GitHub UI).
* Seeing the rendered PR requires extra work in the UI.
* No template is available in the UI for the initial file content analogous to the new issue template.
