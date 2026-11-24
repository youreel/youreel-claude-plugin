# youreel for Claude

Ask Claude about your Instagram, and about the public X account and Telegram channel you
monitor in youreel. This plugin connects Claude to [youreel](https://youreel.app) and adds 4
skills: a daily update, a weekly or monthly report, a comparison with Instagram accounts you
name, and an audit of your latest reels or posts. The daily update, the report and the audit
also cover X and Telegram, each network in its own section.
Claude answers with real numbers from youreel, and tells you when there isn't enough data yet.

## What you need

- A youreel account with your Instagram connected. Sign up at
  [youreel.app](https://youreel.app).
- For X or Telegram: 1 public X account and 1 public Telegram channel you monitor, added on
  the monitor page in youreel.
- A Claude app that supports plugins.

## Connect and disconnect

The plugin adds the youreel connector (`https://www.youreel.app/api/mcp`). The first time
Claude uses it, you sign in to youreel and approve access. The plugin holds no
password, key or token.

To disconnect, open youreel, go to **Settings**, find **Connected AI apps** and
choose **Revoke**. You can also remove the plugin in Claude.

## Skills

| Skill | What you ask | What it reads from youreel |
|---|---|---|
| `daily-update` | "What changed since yesterday?", "which reel is growing fastest?", "how did my X do yesterday?" | Your net follower change since the day before, the last complete day's reach as of the newest capture against the capture before it, posts since the day before, and which of the recent reels it checks (up to 15, excluding posts too new to compare) gained the most views in their last day. |
| `account-report` | "Give me my weekly report", "how did my Instagram do the last 4 weeks?", "how did my X do this week?" | Your followers, the posts you published, their reach, views, saves and shares against the period before, your best and weakest post, and youreel's insights. |
| `compare-accounts` | "Compare me with @a and @b", "who's growing faster, me or @a?" | Followers, posts per week, and average likes and comments for you and up to 5 accounts you name, over the same 56 days, plus follower growth between the readings youreel has in that window. Or 1 account's public profile |
| `post-audit` | "Audit my last 10 reels", "which of my reels worked best?", "which of my Telegram posts worked best?" | Your latest reels (10 unless you ask for more, up to 50) with reach, skip rate, share rate and save rate. Where youreel returned a median, how that reel compares with your other recent reels. |

A week is the last 7 days and a month is the last 4 weeks, not a calendar month.

### X and Telegram

The daily update, the report and the audit also cover the X account and Telegram channel
you monitor, each in its own section:

- **Daily update:** the net follower or subscriber change, new posts, and which recent posts
  gained the most views in their last day.
- **Report:** the follower or subscriber change, how many posts went out, and the best and
  weakest post by views.
- **Audit:** your last posts ranked by views, likes, reposts and replies on X, or by views,
  forwards and reactions on Telegram.

X views are impressions as X reports them. youreel keeps updating an X post's numbers for 5
days after it's posted, and a Telegram post's numbers for 3 months. Numbers from different networks
are never mixed, because each network counts them its own way. `compare-accounts` is
Instagram only.

## What this plugin runs and sends

- **Nothing runs on your computer.** The plugin is 4 skill files, the connector
  address and an icon. No scripts, no programs, no packages.
- **The skills only call youreel's own tools,** through the connector you approved.
  They send nothing anywhere else.
- **The tools only read.** They can't post, edit or delete anything on Instagram, X,
  Telegram or in youreel, and they can't see your Instagram password or access token.
- **Adding an X account or a Telegram channel happens in youreel,** on the monitor page,
  not through Claude. So does removing one.
- **Comparing with an account youreel hasn't read before** makes youreel look up that
  account's public numbers from Instagram. This works only for public Business and
  Creator accounts. youreel keeps reading that account's public numbers from then on.
  It doesn't add the account to your tracked accounts.

## Links

- youreel: [youreel.app](https://youreel.app)
- Privacy policy: [youreel.app/privacy](https://youreel.app/privacy)
- Terms: [youreel.app/terms](https://youreel.app/terms)
- Support: [hello@youreel.app](mailto:hello@youreel.app)

## License

MIT. See [LICENSE](LICENSE).
