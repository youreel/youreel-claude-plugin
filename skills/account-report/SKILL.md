---
name: account-report
description: >-
  A weekly or monthly report from youreel on the user's Instagram and the X
  account and Telegram channel they monitor. Instagram: follower change, posts,
  reach, views, saves and shares against the period before, best and weakest
  post, and youreel's insights. X and Telegram: follower or subscriber change,
  posts, and best and weakest post. Use this whenever the user asks for a weekly
  or monthly report, a recap, "how did my account do this week/month", "how did
  my X do", "how many subscribers did my channel gain", "which of my posts did
  best this week", or "how am I growing compared with last week", even if they
  don't say "report". Questions about a period belong here. Needs the youreel
  connector. Not for comparing with accounts they name (use compare-accounts),
  ranking the last n reels or posts (use post-audit), or what changed since
  yesterday (use daily-update).
---

# Account report

A short, honest report on the user's Instagram, and on the X account and Telegram
channel they monitor in youreel, built only from what the youreel tools return.
The reader is a creator who wants to know what moved and what to do next. A thin report with real numbers is a good outcome. A full report
with 1 invented number is a bad one, because the creator can't tell which number
is the invented one.

## 1. Pick the period

Take the period from the request.

- **Weekly** (the default when the user doesn't say): the last 7 days against the
  7 days before.
- **Monthly**: the last 4 weeks against the 4 weeks before.

youreel's weeks are rolling 7-day windows that count back from now, not calendar
weeks. So a "monthly" report covers the last 28 days, not a calendar month. Say
"the last 7 days" or "the last 4 weeks" in the report, never "this week", "October"
or "this month", so the reader knows exactly which days the numbers cover. If the
user asks for a specific calendar month, explain this and offer the last 4 weeks.

**Networks.** By default the report covers Instagram, X and Telegram, each in its
own section. If the user asks about 1 network ("how did my X do this week"),
report only that one.

## 2. Read the definitions

If the `youreel://definitions` resource is available, read it first. It explains
how to read every number below: what an absent metric means, why `reach` and
`reach28d` answer different questions, and the post floor below which youreel gives
no verdict.

## 3. Call the tools, in this order

1. `get_my_account`: the handle and the latest follower count with its date.
2. `get_my_insights`: youreel's own ranked readings. Each one names the rule that
   produced it.
3. `get_my_trends` with `weeks: 2` for weekly or `weeks: 8` for monthly.
4. `get_my_content` with `since` set to now minus 7 days (weekly) or now minus 28
   days (monthly), as a full UTC timestamp (`YYYY-MM-DDTHH:MM:SSZ`), not a
   calendar date. A bare date starts at midnight UTC and pulls in posts from the
   bucket before. `limit: 50`. If `nextCursor` isn't null, call again with
   `cursor` set to it, and stop only when `nextCursor` is null.
5. `get_content_detail` for the best and the weakest post (see section 4).
   Pass each post's `id` as `contentId`.

Use only these tools for the Instagram part (section 5 adds the X and Telegram
tools). If a call fails, say which one and what is missing as a result, rather
than filling the gap.

## 4. Work out the numbers

**Followers.** From `get_my_trends` `followers.points` (keyed on the day youreel
measured the account), take the newest point and the newest point on or before the
start of the period. Report both values and both dates, and the difference. If no
point is that old, say the follower history doesn't reach back that far yet. Don't
use `followers.change` for this: it spans every returned point, not the period.
Ignore `followers.estimated`: it's youreel's projection, not a measurement.

**What was posted, and how it did.** From `get_my_trends` `weeks`:

- Weekly: compare bucket `index: 0` (the last 7 days) with `index: 1`.
- Monthly: add up buckets 0 to 3 for the last 4 weeks, and 4 to 7 for the 4 before.
  A bucket with `posts: 0` has no metrics because nothing went out that week. It
  adds nothing, and that isn't a gap. A bucket with posts but without a metric is
  a gap for that metric: say that metric wasn't recorded for the whole period
  instead of adding up the buckets that have it.

`posts` counts every post published in that window. A present `reach`, `views`,
`saves`, `shares` or `follows` is the sum of the posts in that bucket that had the
metric recorded, as of now. It can leave some of those posts out. `follows` is
never recorded for reels, so it stays absent on a reels-only account and, on a
mixed account, covers only feed and story posts. Don't call a present number the
total for every post in the bucket. Two things follow, and the report should say
them when they matter:

- These aren't account-wide reach or views for those days. They belong to the
  posts that went out in the window and had the metric.
- The newer posts have had less time to collect views. A smaller number for the
  last 7 days can mean "not finished yet" rather than "worse".

**Best and weakest post.** From the posts `get_my_content` returned for the period,
rank by `reach` and name it as the metric ("by reach"). Leave out posts without
`reach` and say how many you left out. With fewer than 3 posts that have `reach`,
don't name a best or weakest: list the posts with their numbers and say it's too
early to call. Otherwise call `get_content_detail` on the top and the bottom post.

Use `decay` only when `points` isn't empty. The curve covers the first 14 days, or
the first 24 hours when `unit` is `hours`. Say views are still climbing only when
the latest point is recent compared with `postedAt` and with now. An uptick at the
end of a curve that stopped earlier doesn't mean the post is still climbing. Note
`unit`.

`vsAverage` compares the post with the account's mean over the last 30 days,
across every format, not with this period alone. Name the metric. `er` is a
fraction (0.064 is 6.4%). `reach` and `saves` are counts. If `value` or `average`
is absent, say it wasn't recorded.

**Insights.** Pick up to 3 from `get_my_insights`, in the order returned. Quote
the claim and name its `rule`. You may explain what the rule looks at. Don't extend
the claim past what it says, and don't write your own insight from the raw numbers
in its place. An empty list is a real answer: no rule had enough evidence yet.

## 5. X and Telegram

The same period, for the X account and the Telegram channel the user monitors in
youreel.

Call `get_x_account` and `get_telegram_channel` with no `handle` or `username`
and `limit: 100`. If the user asked about 1 network, call only that one. Posts come newest first. Repeat with `offset` set to
`nextOffset` until the oldest post returned is older than the period, or
`nextOffset` is null.

**When a call fails.**

- `not-found`: the user doesn't monitor an account on that network. If they asked
  about that network, say so, and that they add one on the monitor page in youreel.
  You can't add it for them. Otherwise leave that network out without comment.
- Anything else: 1 line naming the tool and quoting its sentence.

**Fetch state.** Every result has `fetching` (`ok` or `stopped`) and `reason`.

- `pending`, or no `profile` and an empty series: youreel hasn't read the
  account yet. Say so in 1 line and skip that network.
- Any other `stopped`: say so first, in plain words, with how fresh the numbers
  are: the newest series date for followers or subscribers, and the newest
  `latest.captured_at` for posts.
  - X `not_found`: X has no account with that handle.
  - X `unavailable`: the account is protected or suspended.
  - X `limit_user` or `limit_global`: youreel's monthly budget for reading X is
    used up, so post counts stopped updating. Fetching starts again by the 1st of
    next month (UTC) at the latest. Followers can still update: check the series
    dates before calling them old.
  - Telegram `not_found`: there's no public channel with that username, or it's
    private.
  - Telegram `session_missing`, `session_dead` or `flood`: a problem on youreel's
    side. The user can't fix it and doesn't need to do anything.
  - `error` on either network: youreel couldn't read it last time. Don't guess
    why.

**Reading the posts.**

- Name a post by its `postedAt` date and the first words of `text`. X posts have
  no link. A Telegram post's `text` can be empty: use its `url`, or only the date
  when `url` is null.
- X posts include the account's replies to other people, and the tool doesn't
  mark which ones are replies. Say "posts and replies" when you count or rank
  them. Reposts of other people's posts aren't included.
- youreel refreshes an X post's counts for 5 days after it's posted. Older posts
  keep their last counts: say "as of" their `latest.captured_at` date, and don't
  measure growth on them.
- Telegram `likes` is all reactions added together. `comments` null means
  youreel has no comment count for that post. It isn't 0. If every post is null,
  the channel has no comments section.
- A `null` count, or a `null` `latest`, wasn't recorded. Write "not recorded",
  never 0.

**Wording.** Write about X and Telegram the way you write about Instagram, each
network in its own section. youreel doesn't check who owns a monitored account,
so say "your" only when the user calls it theirs. Otherwise name it: "@handle on
X", "@channel on Telegram". Never mix networks in 1 table or ranking: X views
are impressions as X counts them, Telegram views are Telegram's own count, and
Instagram reach is a different measure again.

**The numbers, for each network.**

- **Followers (X) or subscribers (Telegram).** From `followerSeries` or
  `subscriberSeries` (1 reading per UTC day, the last one that day), take the
  newest point and the newest point on or before the start of the period, with
  both dates and the difference. It's a net change, as on Instagram. History
  starts the day youreel first read the account. If no point is that old, say
  the history doesn't reach back that far yet.
- **Published in the period**: how many posts (on X, posts and replies), by
  `postedAt`. On Telegram, also give the count for the period before: youreel
  reads 3 months of posts. On X, don't: youreel's first read reached back only up
  to 30 days, so the period before can be incomplete. Don't add up views across
  posts on either network: newer posts have had less time, and X posts stop
  updating after 5 days, so the totals wouldn't line up.
- **Best and weakest by views**, each with its other counts. X: likes, reposts,
  replies, quotes, bookmarks. Telegram: forwards, likes, comments. Leave out
  posts without `views` and say how many. With fewer than 3 posts that have
  views, list them instead and say it's too early to tell. In a 7-day period the
  newest posts have had less time: say so when it affects which post is best.
- **Not available** for X or Telegram: reach, saves, youreel's insights and who
  follows or subscribes. If the user asks, say youreel doesn't have them there.

## 6. Write the report

Keep it readable in under a minute. Use this shape, and drop any section the data
can't fill rather than padding it:

```
# @handle: the last 7 days        (or: the last 4 weeks)

Followers: 1,480 on 2026-11-16, up 32 from 1,448 on 2026-11-09.
Posts published: 4 (5 in the 7 days before).

| | Last 7 days | 7 days before |
|---|---|---|
| Reach | ... | ... |
| Views | ... | ... |
| Saves | ... | ... |
| Shares | ... | ... |

Best post by reach: <date, format, first words of the caption>, <numbers>.
Weakest post by reach: ...

What youreel noticed:
- <claim> (rule: <rule>)

Next step: <1 step, and the number it rests on>

## @handle on X
Followers: 1,204 on 2026-11-16, up 18 from 1,186 on 2026-11-09.
Posts and replies published: 6.
Best post by views: 2026-11-12, "<first words>", 3,410 views, 52 likes (as of 2026-11-16).
Weakest post by views: ...

## @channel on Telegram
Subscribers: ...
Posts published: ...
Best post by views: ...
Weakest post by views: ...
```

Write in plain, direct second person ("you posted 4 times"). Say "not recorded" for
a missing metric, never 0 or a dash that reads as 0.

Add a next step only when an insight's `action` or a clear number in the report
supports it, and name what it rests on. If nothing supports one, leave it out.

## Rules that keep the report honest

- Quote only numbers the tools returned, or a sum or difference of them that you
  show. Don't average, estimate, extrapolate or fill a gap.
- A missing metric was never recorded. It isn't zero. Say it wasn't recorded, and
  why if you know: Instagram withholds several metrics for accounts under roughly
  1,000 followers.
- Below the post floor in the definitions (3 posts), there's no verdict. Say it's
  too early to tell.
- Don't compare `reach` with `reach28d`. They answer different questions.
- Don't explain a post's result by its caption, hook, audio, thumbnail or posting
  time unless a tool returned something that supports it. You can't see the video.
- The tools only read. If the user asks you to change something, tell them where to
  do it in youreel rather than implying you did it.
- If the account has no posts in the period, or the tools say there's no connected
  account, say so plainly and stop the Instagram part. Still add X and Telegram
  (section 5) unless the user asked about Instagram only. Don't build a report from other periods to fill the space, don't
  guess why the data is missing beyond what a tool said, and don't promise when
  more data will arrive.
