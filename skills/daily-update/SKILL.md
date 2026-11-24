---
name: daily-update
description: >-
  A short daily overview from youreel of the user's Instagram and of the X
  account and Telegram channel they monitor in youreel: the net follower or
  subscriber change since the day before, Instagram's last complete day of reach,
  posts published since the day before, and which recent reels or posts gained
  the most views in their last measured day. Use this whenever the user asks for
  a daily update, a daily overview, "what changed since yesterday", "how did
  yesterday go", "anything new on my Instagram today", "how did my X do
  yesterday", "which reel or post is growing fastest right now", or a morning
  check-in, even if they don't say "daily". Needs the youreel connector. Not for
  a weekly or monthly report (use account-report), not for ranking a set of
  reels or posts overall (use post-audit), and not for other people's Instagram
  accounts (use compare-accounts).
---

# Daily update

A quick read on what moved since yesterday on Instagram, X and Telegram, built
only from what the youreel tools return. If the user asks about 1 network only,
cover only that one. The reader checks it daily, so keep it short and say plainly
when a day moved too little to mean anything. 1 day is a small sample.

What this skill doesn't cover: likes, saves, shares or comments gained since
yesterday. On Instagram the tools return those only as current totals. If the
user asks, say youreel doesn't show that day by day yet. On X and Telegram,
`get_x_post` and `get_telegram_post` return every count per day, so answer from
there when the user asks about 1 post.

## 1. Read the definitions

If the `youreel://definitions` resource is available, read it first. It explains
how to read every number below: what an absent metric means, why a follower change
is a net change, and why `reach` and `reach28d` answer different questions.

## 2. Find the update day

You usually don't know the exact time, and no tool returns it. So anchor the update
on the data, not the clock. Call `get_my_account` (for the handle), then
`get_my_trends` with `weeks: 1`. The **update day** is the date of the newest point
in `followers.points`. If there are no follower points, use the newest date in
`reach.points`. If neither has a point, say youreel has no daily data for the
Instagram account yet, go to section 5, and write only the X and Telegram
sections.

If you know today's date and the update day is more than 1 day before it, say how
old the newest data is.

## 3. Call the rest, in this order

1. `get_my_content` with `since` set to the day before the update day, as a UTC
   timestamp at midnight (`YYYY-MM-DDT00:00:00Z`), and `limit: 50`. These are the
   new posts: everything since the start of that day. If `nextCursor` isn't null,
   call again with `cursor` set to it, and stop only when `nextCursor` is null.
2. `get_my_content` with `contentType: "reel"`, `since` set to 14 days before the
   update day (same format) and `limit: 50`. If `nextCursor` isn't null, call
   again with `cursor` set to it, and stop only when `nextCursor` is null.
3. `get_content_detail` on each reel from step 2 that isn't one of the new posts,
   newest first, passing its `id` as `contentId`. Stop at 15 reels so the update
   stays quick, and say how many older ones you left out.

Use only these tools for the Instagram part (section 5 adds the X and Telegram
tools). If a call fails, say which one and what is missing as a result, rather
than filling the gap.

## 4. Work out the numbers

**Followers.** From `followers.points` (1 point per UTC day youreel measured the
account), take the newest point and the point before it. Report both values, both
dates, and the difference. If the 2 dates aren't consecutive days, say how many days
the change covers. With fewer than 2 points, say there's no follower change to show
yet. Call it a net change ("+12 followers since 2026-11-16"). It can't tell new
followers from unfollows, so never write "new followers", "unfollows" or "lost
followers". Ignore `followers.estimated`: it's youreel's projection, not a
measurement.

**Reach.** From `reach.points`, take the newest point and the one before it.
Each point's date is the day youreel captured the account. The number is the
last complete day's reach as of that capture, not reach during the capture day.
A point dated 2026-11-17 holds the complete day before that capture. Followers
on the same date are the count at capture time, so don't date reach and followers
as if both happened on the capture day. Report both reach values and both capture dates, and
say each figure is the last complete day as of that capture. The difference
between 2 consecutive captures is still 2 consecutive complete days.

The newest point is always a stored reading. Quote it, including a 0. A 0 that
isn't the first or last point can be a gap the tool filled with 0, and you can't
tell that fill from a real 0. Don't report that interior 0 as a drop. Say there's
no reach reading for that day. Don't compare `reach` with `reach28d`.

**New posts.** List each new post with its format, `postedAt`, the first words of
its caption, and the numbers it has so far (`reach`, `saves`, and for a reel
`skipRatePct`, `shareRatePct`). Age each one from `postedAt` against the update
day and against today. Say a post is at most 1 or 2 days old only when those
timestamps show that. Otherwise say how old it is. They're still collecting.
Don't rank them against older posts.

**Fastest growing reels.** For each reel you called `get_content_detail` on:

- Use `decay` only when `points` isn't empty and `unit` is `days`. Its `age` is
  whole days since the reel was posted, not calendar days, and its value is views.
- The latest point was captured about `age` days after `postedAt`. Use the curve
  only if that lands on the update day or the day before. An older latest point
  means youreel has no recent reading for it.
- Take the latest point and the point 1 day of age before it. Views gained is the
  difference. If there's no point exactly 1 day before, leave the reel out.
- The curve stops at day 14, so a reel near that age may have no recent point.

Rank the reels that have a views-gained figure, highest first. This is the top of
the reels you checked (at most 15), not every reel from the last 14 days, and it
leaves out the new posts, which have no earlier point to subtract. Show the top 3
with both readings ("from 3,707 views on day 5 to 4,031 on day 6, +324"). Say how
many reels you didn't check. List the ones you checked but couldn't compare, and
why, in 1 line. When the top reels gained similar amounts, say they're close
rather than naming 1 as fastest.

## 5. X and Telegram

The same update for the X account and the Telegram channel the user monitors in
youreel.

Call `get_x_account` and `get_telegram_channel` with no `handle` or `username`
and `limit: 100`. If the user asked about 1 network, call only that one. Each network has its own **update day**: the newest date in its
`followerSeries` or `subscriberSeries`. Posts come newest first. Repeat with
`offset` set to `nextOffset` until the oldest post returned is older than the
growth window below, or `nextOffset` is null.

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

- **Followers (X) or subscribers (Telegram).** From the series (1 reading per UTC
  day, the last one that day), take the newest point and the point before it,
  with both dates and the difference. If the dates aren't consecutive, say how
  many days it covers. With fewer than 2 points, say there's no change to show
  yet. It's a net change, as on Instagram.
- **New posts**: posts whose `postedAt` is on or after the start of the UTC day
  before the update day, with their counts so far as of `latest.captured_at`.
  X: views, likes, reposts, replies. Telegram: views, forwards, likes, comments.
  They're still collecting, so don't rank them.
- **Growing fastest**: the same idea as the reels above. Take the posts that
  aren't new and were posted in the 5 days (X) or 14 days (Telegram) before the
  update day. On X that's every post youreel still updates. On Telegram, 14 days
matches the reels above. Newest first, call
  `get_x_post` (with the post's `id` as `postId`) or `get_telegram_post` (with
  its `messageId`) on up to 10 per network, and say how many you left out. From
  `daily` (1 reading per UTC day), use a post only if its newest point is on the
  update day or the day before, and there's a point for the day before that.
  Views gained is the difference. Show the top 3 with both readings and dates.
  When they're close, say so rather than naming 1 as fastest. List the posts you
  checked but couldn't compare, and why, in 1 line (for example, only 1 reading
  so far).

## 6. Write the update

Keep it readable in 30 seconds. Use this shape, and drop any section the data
can't fill:

```
# @handle: 2026-11-17 against the day before

Followers: 1,475 on 2026-11-17, +12 from 1,463 on 2026-11-16.
Reach: 3,210, the last complete day as of the 2026-11-17 capture, against 2,980 as of the 2026-11-16 capture.

New since the start of 2026-11-16 (UTC):
- <format>, <postedAt>, "<first words of caption>": <numbers so far>.

Growing fastest among the reels checked (views gained in their last day):
1. <reel>: +<n> (from <a> on day <x> to <b> on day <y>)
2. ...
Not compared: <reels and why>.

## @handle on X
Followers: 1,204 on 2026-11-17, +3 from 1,201 on 2026-11-16.
New: <postedAt>, "<first words>": <views> views, <likes> likes so far.
Growing fastest (views gained in their last day):
1. <post>: +<n> (from <a> on <date> to <b> on <date>)

## @channel on Telegram
Subscribers: 640 on 2026-11-17, +2 from 638 on 2026-11-16.
...
```

Start with the update itself, with no opener like "Here's your update". Write in
plain, direct second person ("your reel from Nov 10"), in short sentences.
Say "not recorded" for a missing metric, never 0. If nothing moved enough to
mention, say so in 1 line. A quiet day is a real answer.

## Rules that keep the update honest

- Quote only numbers the tools returned, or a difference of 2 of them that you
  show. Don't average, estimate, extrapolate or fill a gap.
- A missing metric was never recorded. It isn't zero. Instagram withholds several
  metrics for accounts under roughly 1,000 followers.
- 1 day is a small sample. Don't call a change of a few followers or a few views
  a trend, and don't give a verdict on a format or a label from 1 day.
- Don't explain a change by a post's caption, hook, audio, thumbnail or posting
  time unless a tool returned something that supports it. You can't see the video.
- If the Instagram account has no data yet, say so plainly, then still add X and
  Telegram (section 5) unless the user asked about Instagram only. Don't guess why, beyond what a tool said, and don't
  promise when data will arrive.
- The tools only read. If the user asks you to change something, tell them where to
  do it in youreel rather than implying you did it.
