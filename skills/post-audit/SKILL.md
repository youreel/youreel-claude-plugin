---
name: post-audit
description: >-
  Audit the user's last n Instagram reels, or the last n posts from the X
  account or Telegram channel they monitor in youreel, and say which ones worked
  better and on which numbers. Reels: reach, skip rate, share rate and save
  rate, and, where youreel returned one, each reel against the median of the
  account's other recent reels. X posts: views, likes, reposts and replies.
  Telegram posts: views, forwards and reactions. Use this whenever the user asks
  to audit, review, rank or compare their recent reels or posts, asks "which of
  my last 10 reels did best", "which of my X posts worked", "what worked on my
  Telegram channel", "why are my reels flopping", or names a number of reels or
  posts to look at, even if they don't say "audit". Needs the youreel connector.
  A question about the last n posts belongs here. Not for a report on a period
  like "this week" (use account-report), for which reel or post grew most since
  yesterday (use daily-update), or for other people's Instagram accounts (use
  compare-accounts).
---

# Post audit

Look at the user's most recent Instagram reels, or the recent posts from the X
account or Telegram channel they monitor, and say which ones did better, on which
numbers, using only what youreel measured. The useful answer names the metric
behind every ranking. "Reel 3 did best" means nothing; "reel 3 had the most reach
and the lowest skip rate" is something the creator can act on.

## 1. Pick the network and n

- **Network**: Instagram reels, unless the user names X or Telegram. If they name
  more than 1, audit each on its own. Sections 2 to 4 are for reels, section 5 is
  for X and Telegram. If they named only X or Telegram, skip the reel calls.
- **n**: the number the user gave. Default to 10. The audit covers at most 50
  reels, or 100 X or Telegram posts. If the user asks for more, say so and audit
  the most it can.

## 2. Read the definitions

If the `youreel://definitions` resource is available, read it first. It explains
the rates below: which are percents and which are fractions, why skip rate is
better when lower, and the order in which Instagram weighs them.

## 3. Call the tools

1. `get_my_content` with `contentType: "reel"` and `limit: n`. It returns the newest
   reels first.
2. `get_content_detail` on each reel when n is 10 or fewer, passing the post's
   `id` as `contentId`. When n is over 10, call it on the top 3 and bottom 3 by
   reach only (section 4), so the audit stays quick.

Use only these 2 tools for reels. If fewer reels come back than n, audit what came
back and say how many that was. If no reels come back, say youreel has no reels for this
account yet and stop. Don't guess why, beyond what a tool said, and don't promise
when they will arrive.

## 4. Rank the reels

Make separate rankings, each named by its metric. Don't merge them into one score:
a reel can lead on reach and trail on skip rate, and that split is the finding.

1. **By reach** (`reach`), highest first.
2. **By skip rate** (`skipRatePct`), lowest first. Lower is better.
3. **By share rate** (`shareRatePct`), highest first.
4. **By save rate** (`saveRatePct`), highest first.

Skip rate, share rate and save rate are percents (41 means 41%). Skip rate is
Meta's own percent. Share rate and save rate are percents of reach (100 × that
count ÷ reach). Instagram weighs skip rate first and share rate second, then like,
save and comment rate, so a clear gap in skip rate matters more than the same gap
in save rate.

For each ranking, list the reels that don't have that metric and say it wasn't
recorded for them. Leave them out of that ranking instead of counting them as 0.

**Against the other reels.** For a reel whose `get_content_detail` returned
`reelMedian`, compare its rates with that object: the median of each rate over the
account's other reels in the last `windowDays` days, excluding this reel, with
`reels` for how many that median is based on. Say "above" or "below the median"
with both numbers. Print a median only for that reel. Don't borrow another reel's
median. A rate with no median, or a reel you didn't call `get_content_detail` on,
has nothing to compare. Say that. Don't treat a missing median as average.

**Age.** A reel posted a day ago hasn't finished collecting reach. Give each reel's
`postedAt`. Use `decay` only when `points` isn't empty. The curve covers the first
14 days, or the first 24 hours when `unit` is `hours`. Say views are still
climbing only when the latest point is recent compared with `postedAt` and with
now. An uptick at the end of a curve that stopped earlier doesn't mean the reel is
still climbing. Note `unit`. Don't rank a reel that is still climbing as the
weakest without saying it's still climbing.

**Labels.** If the reels carry labels (the creator's own words for what a post is
about), you may group the reels by label and show each group's numbers. With fewer
than 3 reels in a group there's no verdict for it. Say it's too early to tell.

## 5. X and Telegram posts

Call `get_x_account` or `get_telegram_channel` with no `handle` or `username` and
`limit: n`. Posts come newest first.

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

**Rank them.** From each post's `latest`, make separate rankings named by their
metric, the same as reels. Don't merge them into one score.

- **X**: by `views`, `likes`, `reposts` and `replies`, highest first. `quotes` and
  `bookmarks` are there too if the user asks.
- **Telegram**: by `views`, `forwards` and `likes`, highest first.

There's no median or rate for these posts. Don't work one out. Leave a post out
of a ranking where that metric is `null`, and say it wasn't recorded.

**Age.** Give each post's `postedAt`. The newest posts are still collecting, so
don't call one the weakest without saying that.

## 6. Write the audit

```
# Your last 10 reels

Which worked better:
- Most reach: <date, first words of caption>, <reach>.
- Lowest skip rate: ..., <skipRatePct>%. Add "(median <median>% over <reels> other reels)" only when this reel's get_content_detail returned reelMedian.skipRatePct.
- Highest share rate: ... (same rule for its median)
- Highest save rate: ... (same rule for its median)

| Posted | Reel | Reach | Skip rate | Share rate | Save rate | vs median |
|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | not compared, unless this reel's detail returned reelMedian |

Not recorded: <which metrics are missing for which reels, and why if known>.

What stands out: <2 or 3 sentences, each with its numbers>.
```

For X or Telegram, use the same shape with that network's metrics and no median
column, titled "@handle on X: the last 10 posts and replies" or "@channel on
Telegram: the last 10 posts". Say "your" only when the user calls it theirs.

Write in plain, direct second person ("your reel from Nov 10"), in short sentences.
Say "not recorded" for a missing metric, never 0.

Mark a reel the creator flagged as a trial (`isTrial: true`) with "(trial)". That
flag is the creator's own marking; Instagram doesn't share trial status, so never
say a reel wasn't a trial.

End with what stands out: a reel that leads on several rankings, a clear split
between reach and skip rate, or a label group that does better, each with the
numbers behind it. If the reels are close on every ranking, say they're close and
that no reel clearly did better. That's a real answer.

## Rules that keep the audit honest

- Quote only numbers the tools returned. Don't average, estimate or fill a gap.
- A missing metric was never recorded. It isn't zero. Instagram withholds several
  metrics for accounts under roughly 1,000 followers, which is the usual reason.
- With fewer than 3 reels or posts, there's no verdict on which worked better. List them
  with their numbers and say it's too early to tell.
- Don't attribute a result to the hook, caption, audio, thumbnail, length or posting
  time unless a tool returned something that supports it. You can't see the video.
  If the user asks why a reel did well, say what the numbers show and that the
  cause isn't in the data.
- `avgPctWatched` isn't a completion rate, and it can go above 100 when people
  watch a reel more than once. If you quote it, say so.
- The tools only read. If the user asks you to change something, tell them where to
  do it in youreel rather than implying you did it.
