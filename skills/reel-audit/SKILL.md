---
name: reel-audit
description: >-
  Audit the user's last n Instagram reels with youreel and say which ones worked
  better and on which numbers: reach, skip rate, share rate and save rate, and,
  where youreel returned one, each reel against the median of the account's other
  recent reels. Use this whenever the user asks to audit, review, rank or compare
  their recent reels, asks "which of my last 10 reels did best", "what worked in
  my last reels", "why are my reels flopping", "which reel should I make more of",
  or names a number of reels to look at, even if they don't say "audit". Needs
  the youreel connector. Not for a whole account report (use account-report),
  for which reel grew most since yesterday (use daily-update), or for other
  people's accounts (use compare-accounts).
---

# Reel audit

Look at the user's most recent reels and say which ones did better, on which
numbers, using only what youreel measured. The useful answer names the metric
behind every ranking. "Reel 3 did best" means nothing; "reel 3 had the most reach
and the lowest skip rate" is something the creator can act on.

## 1. Pick n

- Use the number the user gave. Default to 10.
- youreel returns at most 50 posts per call, so the audit covers at most 50 reels.
  If the user asks for more, say so and audit the last 50.

## 2. Read the definitions

If the `youreel://definitions` resource is available, read it first. It explains
the rates below: which are percents and which are fractions, why skip rate is
better when lower, and the order in which Instagram weighs them.

## 3. Call the tools

1. `get_my_content` with `contentType: "reel"` and `limit: n`. It returns the newest
   reels first.
2. `get_content_detail` on each reel when n is 10 or fewer, passing the post's
   `id` as `contentId`. When n is over 10, call it on the top 3 and bottom 3 by
   reach only (step 4), so the audit stays quick.

Use only these 2 tools. If fewer reels come back than n, audit what came back and
say how many that was. If no reels come back, say youreel has no reels for this
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

## 5. Write the audit

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
- With fewer than 3 reels, there's no verdict on which worked better. List them
  with their numbers and say it's too early to tell.
- Don't attribute a result to the hook, caption, audio, thumbnail, length or posting
  time unless a tool returned something that supports it. You can't see the video.
  If the user asks why a reel did well, say what the numbers show and that the
  cause isn't in the data.
- `avgPctWatched` isn't a completion rate, and it can go above 100 when people
  watch a reel more than once. If you quote it, say so.
- The tools only read. If the user asks you to change something, tell them where to
  do it in youreel rather than implying you did it.
