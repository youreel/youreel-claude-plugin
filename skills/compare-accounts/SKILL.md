---
name: compare-accounts
description: >-
  Compare the user's own Instagram account with up to 5 other Instagram accounts
  they name, side by side, using youreel: followers, follower growth, posts per
  week, average likes and comments, and engagement rate over one shared window.
  Use this when the user gives 1 or more Instagram handles and wants to know how
  they stack up, asks "compare me with @someone", "how do I compare to these
  creators", "am I posting more than @x", "who's growing faster, me or @x", or
  wants a quick look at another account's profile, even if they don't say
  "compare". Use it with no handles only when they explicitly ask to compare
  against the accounts they already track in youreel. Needs the youreel
  connector. Not for a report on the user's own account, and not for "how am I
  growing compared with last week or last month" (use account-report).
---

# Compare accounts

Put the user's account beside the accounts they name, on numbers both sides
genuinely have. youreel can read only what Instagram makes public for an account
the user doesn't own, so the comparison is deliberately narrow: followers, growth,
how often they post, and average likes and comments. Reach, views, saves and shares
don't exist for other people's accounts, so they are never part of this.

## 1. Collect the handles

- Take the handles from the request. Strip a leading `@`. A profile URL also works.
- **More than 5 handles:** ask the user which 5 to compare. Don't pick for them and
  don't split them across several calls, because each call uses 1 shared window
  and mixing results from 2 calls invites mixing 2 windows.
- **No handles:** use this skill only when the user explicitly asks to compare
  against the accounts they already track. Then call `compare_accounts` with no
  `handles`, and say that's what you compared against. A question about their own
  account versus last week or last month, with no handles, is `account-report`,
  not this skill.
- **One handle and a question about that account only** ("what's @x like?",
  "how many followers does @x have?"): call `lookup_account` with that handle
  instead of running a comparison.

Handles passed to `compare_accounts` replace the tracked list for that call only.
They don't add anything to the user's tracked accounts and use none of their
tracking slots.

## 2. Read the definitions

If the `youreel://definitions` resource is available, read it first. It explains
what an absent number means and why it's never zero.

## 3. Call the tool

Call `compare_accounts` with `handles` set to the list (or with no `handles`, as
above). Use only `compare_accounts`, or `lookup_account` for a single-profile
question. A handle youreel has never read before is looked up on the spot, so a
first comparison can take several seconds per new handle. That's expected.

If the call fails, quote the tool's own sentence. Don't add a cause it didn't give.

- `not-found`: say the handles couldn't be read, in the tool's words. Don't add
  that an account is private, personal, or unreachable unless the tool said that.
- `invalid-input`: quote the tool's `detail`. Don't assume which part of the
  request it rejected.
- `throttled`: say to try again after `retryAfterSeconds`. Don't describe this as
  a daily limit on new accounts. The tool's sentence is the reason.
- `tier-required`: this tool isn't included in the user's subscription. Say so
  and give the `upgradeUrl` it returned.
- `billing-blocked`: the subscription has ended and reads are paused. Say so and
  give the `upgradeUrl` it returned. Don't say compare isn't included.
- `no-account`: the user hasn't connected Instagram in youreel yet.

## 4. Write the comparison

State the window first: "over the last `windowDays` days". That window covers
posts per week, likes and comments. It doesn't by itself date the growth figure
(see below). Then one table, the user's account (`own`) first, then each account
in `accounts`:

```
Over the last 56 days:

| Account | Followers | Growth | Posts per week | Avg likes | Avg comments | Engagement rate |
|---|---|---|---|---|---|---|
| @you (you) | ... | ... | ... | ... | ... | ... |
| @them | ... | ... | ... | ... | ... | ... |

<the tool's erDefinition, unchanged>
```

How to read the fields:

- `followersGrowth` and `engagementRate` are fractions: 0.031 is 3.1%. Show them as
  percents.
- `followersGrowth` is the change from the first follower reading in the window to
  the last. The tool doesn't say how far apart those readings are. Don't say the
  account grew that much over all `windowDays` days.
- `postsPerWeek` is the figure normalized to the whole window. 0 is real: that
  account posted nothing in the window.
- Every other absent field wasn't recorded. Write "not recorded", never 0. A
  missing `followersGrowth` means youreel has fewer than 2 follower counts for that
  account in the window, which is normal for an account it only started reading
  today. Say "not measured yet", not "no growth".
- An account with `status` other than `active` (for example `unavailable`) stopped
  updating, usually because it went private or was removed. Say its numbers are its
  last known ones.
- If `own` is missing, the user hasn't connected Instagram, so there's nothing to
  compare against. Show the other accounts and say why the user's row is missing.

After the table, list every handle in `unavailable`, by name. The tool gives no
reason for each one, so don't guess private, personal, or unreachable. Say youreel
couldn't read them. Never compare fewer accounts than the user asked for without
saying which ones dropped out.

Then, in 2 or 3 plain sentences in second person ("you post 4.6 times a
week"), say what the numbers show: who posts more often, who grew faster between
the readings youreel has, whose posts get more likes and comments per follower.
Quote the numbers. Where 2 accounts are close, say they're close rather than naming a
winner.

## Rules that keep the comparison honest

- Quote only numbers the tools returned. Don't estimate a competitor's reach, views
  or saves: youreel can't see them, and nor can you.
- Don't put this engagement rate in the same sentence as a per-post engagement rate
  from another youreel tool. They use different denominators. The tool says so in
  `erDefinition`; repeat that line under the table.
- Don't judge content quality, style or strategy. The numbers say how often an
  account posts and how much it is liked and commented on, not why.
- Don't explain a difference by posting time, captions, audio or topics unless a
  tool returned something that supports it.
- The tools only read. Comparing doesn't follow, track or contact any account.
