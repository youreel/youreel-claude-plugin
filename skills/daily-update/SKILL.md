---
name: daily-update
description: >-
  A short daily overview of the user's own Instagram account from youreel: the
  net follower change since the day before, the last complete day's reach as of
  the newest capture against the capture before it, posts published since the
  day before, and which of the recent reels it checks gained the most views in
  their last measured day. Use this whenever the user asks for a daily
  update, a daily overview, "what changed since yesterday", "how did yesterday
  go", "anything new on my Instagram today", "which reel is growing fastest right
  now", or a morning check-in, even if they don't say "daily". Needs the youreel
  connector. Not for a weekly or monthly report (use account-report), not for
  ranking a set of reels overall (use reel-audit), and not for other people's
  accounts (use compare-accounts).
---

# Daily update

A quick read on what moved since yesterday, built only from what the youreel tools
return. The reader checks it daily, so keep it short and say plainly when a day
moved too little to mean anything. 1 day is a small sample.

What this skill doesn't cover: likes, saves, shares or comments gained since
yesterday. The tools return those only as current totals, with no day-by-day
history. If the user asks, say youreel doesn't show that day by day yet.

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
account yet and stop.

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

Use only these tools. If a call fails, say which one and what is missing as a
result, rather than filling the gap.

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

## 5. Write the update

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
- If the account has no data yet, say so plainly and stop. Don't guess why, beyond
  what a tool said, and don't promise when data will arrive.
- The tools only read. If the user asks you to change something, tell them where to
  do it in youreel rather than implying you did it.
