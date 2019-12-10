# RFC 37: Split META.yml `suggested_reviewers` into `suggested_reviewers` and `notify`

## Summary

Change `META.yml` files in wpt to have `notify` in addition to `suggested_reviewers`.
@wpt-pr-bot mentions everyone in `notify` and then requests review from one person from `suggested_reviewers`.

## Details

At TPAC 2019 we set out some [priorities for 2020](https://docs.google.com/document/d/1gie7LFb6cAUfabY86MYuWM7m7ux_FaKhDkLdpz0zWkg/edit),
which include increasing PR review velocity and reduce the number of stale PRs.

wpt has META.yml files with `suggested_reviewers`. @wpt-pr-bot will request review from everyone listed in `suggested_reviewers` for PRs that change that directory, and also assign one of them in a round-robin fashion.

See https://web-platform-tests.org/reviewing-tests/index.html#notifications

The documentation only says that `suggested_reviewers` is for getting notifications for PRs.

This has a few problems:

* There is a disconnect between what contributors would reasonably expect and what often actually happens. When people are requested review and someone is assigned, it communicates that the person assigned will help the contributor to get it landed.
* The current system doesn't support varying preferences from reviewers. Some people want to see all PRs for a given folder (which is supported by the current system), while others want to help with reviews but not be notified for everything, and some may want to be notified for everything but can't commit to helping with reviews.

To address these problems, have two separate keys in `META.yml`:

* `suggested_reviewers`: @wpt-pr-bot will only request review from one of these, similar to how assignee currently works.
* `notify`: @wpt-pr-bot will add a comment (before requesting reviews) notifying everyone listed.

Either @wpt-pr-bot can continue to also assign the selected reviewer, or it can only request review and not assign anyone. (This should be decided before merging this RFC.)

This setup allows the following combinations:

* People who want to help with review and want to be notified for everything add themselves to both lists.
* People who want to help with review but not be notified for everything add themselves to `suggested_reviewers`.
* People who don't want to help with review but still be notified for everything add themselves to `notify`.

## Risks

Having bots add comments in wpt has been a source of contention in the past. This RFC reintroduces a comment.

The known problems are:

* Additional notifications. When someone is requested review, and a bot later adds a comment, it can cause a new notification.
* Clutter the thread with bot comments.

These are partially mitigated by having the bot add the comment before (or at the same time as) requesting review. The bot could further keep the comment short, and edit its first comment instead of adding more comments if the PR is updated.

## Alternatives considered

* Encourage reviewers to use email filters. Not everyone uses email for GitHub in the first place, and most likely not everybody is fond of setting up email filters for wpt.
