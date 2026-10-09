# x-review

A Claude skill that scores a draft X (Twitter) post from **1 to 10** using the weights from X's open-source For You ranker, tells you which actions the post will earn, and rewrites it until it scores **9 or higher**.

The idea comes from a prompt Noah Kagan (AppSumo) shared: score the post using X's weights, say which actions it will earn, then rewrite it to earn a share or a reply. This skill turns that into a repeatable workflow.

## What it does

Paste a draft and type `/x-review`. You get back, in this order:

1. **Score: X/10** for the draft.
2. **Likely to earn:** the actions the post will probably trigger, with the weight in brackets, highest first.
3. **One fix:** the single biggest leak costing points. Not a list of ten.
4. **Rewrite (Y/10):** a paste-ready version.
5. **Why it scores higher:** one or two lines.
6. **Posting note:** one practical line (for example "put the link in a self-reply").

The skill re-scores its own rewrite with the same rubric and only shows you the final score.

## The weights it uses

X's For You feed scores each post as a weighted sum of predicted actions: `Final Score = sum(weight x predicted probability of that action)`. The values below are the defaults in [`xai-org/x-algorithm`](https://github.com/xai-org/x-algorithm), `home-mixer/params/param.rs`. Weights can change, so re-check the repo if you suspect drift.

### Positive weights

| Action | Weight | Equivalent likes |
|---|---|---|
| Share via copy link | 20.0 | 40 |
| Reply | 5.0 | 10 |
| Quote | 5.0 | 10 |
| Share via DM | 5.0 | 10 |
| Follow author | 4.0 | 8 |
| Share (generic) | 2.0 | 4 |
| Repost | 1.0 | 2 |
| Like | 0.5 | 1 |

Small weights (post click 0.4, open link 0.2, video open 0.07, photo expand 0.05, dwell 0.05) are ignored when writing. Bookmark has no weight in `param.rs`, so it is not counted.

### Mutual follow boost

When viewer and author follow each other, a reply gets an extra +15, so it is worth 20, the same as a copy link.

### Negative weights

| Action | Weight |
|---|---|
| Report | -234.0 |
| Mute author | -58.8 |
| Not interested | -43.2 |
| Block author | -31.2 |

These scale predicted probabilities, not raw counts, so one report does not cancel hundreds of likes. But outrage bait, spam, and fight-picking are expensive if the model predicts even a small chance of them.

### Adjustments after scoring

- **Author diversity:** each extra post from the same author in a batch is discounted (0.5 decay, 0.25 floor). Do not post twice in a row.
- **Out-of-network:** posts from accounts the viewer does not follow are multiplied by 0.75 (0.5 for topic-based).
- **Low-impression boost:** small accounts get a boost toward a target position.
- **48 hour cutoff:** posts older than 48 hours are filtered before scoring.

Full detail is in [`x-review/references/weights.md`](x-review/references/weights.md).

## How the 1 to 10 score works

The score is the sum of five parts, rounded to the nearest half point. It is strict: most first drafts land at 4 to 6.

| Part | Points | What it measures |
|---|---|---|
| Hook | 0 to 2 | Does the first line stop the scroll with a specific claim, number, or tension? |
| Standalone value | 0 to 3 | Could someone paste this link to one person with no context? Drives copy link and DM share. |
| Reply or quote trigger | 0 to 2 | Is there an easy, natural reason to answer or add a take? |
| Follow signal | 0 to 1 | Does it show a clear, repeatable theme that makes people want more? |
| Clean delivery | 0 to 2 | Deductions for a link in the body, hashtags, engagement bait, walls of text, multiple ideas, outrage bait. |

### Caps

- Standalone value of 1 or less: the post cannot score above **7**.
- Reply or quote trigger of 0: the post cannot score above **8**.
- Link in the body with standalone value under 3: the post cannot score above **6**.

### Calibration

| Score | Meaning |
|---|---|
| 9 to 10 | Built to earn a copy link or a reply. Specific, complete, easy to answer. |
| 7 to 8 | Good post, one clear fix left. |
| 5 to 6 | Fine content, not built for the heavy actions. Typical first draft. |
| 3 to 4 | Generic or announcement style. Earns mostly likes. |
| 1 to 2 | Self-promo, link dump, or vague. |

The score is a prediction of what the post is *built* to earn. It is not a guarantee of views, and the skill never claims one. Full rubric: [`x-review/references/rubric.md`](x-review/references/rubric.md).

## Rewrite patterns

Rewrites follow one of five patterns, chosen to fit the draft's real content:

- **A. Sendable framework** (copy link, DM share)
- **B. The A or B reply** (reply)
- **C. Position people add to** (quote)
- **D. Story with a question** (reply, follow)
- **E. Screenshot or proof post** (copy link, DM share)

Hard rules: no em dashes, no invented numbers (gaps become `[brackets]` for you to fill), no engagement begging, no hype words, and the user's facts are never changed. See [`x-review/references/rewrite-patterns.md`](x-review/references/rewrite-patterns.md).

## Install

### Claude.ai and Claude desktop

1. Download `x-review.skill` from the [v1.0.0 release](../../releases/tag/v1.0.0).
2. Open **Settings > Capabilities > Skills** (on some plans: **Customize > Skills**).
3. Click **Upload skill** and select `x-review.skill`.
4. Make sure the skill is toggled on.

### Claude Code

Clone the repo and copy the folder into your skills directory:

```bash
git clone https://github.com/swayamdhawan/x-review-skill.git
cp -R x-review-skill/x-review ~/.claude/skills/x-review
```

For a single project instead of globally, copy it to `.claude/skills/x-review` inside that project. Restart Claude Code if it was already running.

## How to use `/x-review`

Type `/x-review` followed by your draft:

```
/x-review
Most coaches cap their income by sending 40 DMs a day. Here is the 3 step version...
```

Or skip the command and just say "score this tweet", "will this post get reach", or paste a draft and ask "thoughts?" in an X context. The skill triggers on those too.

- If you run `/x-review` with no post, it asks for one.
- Ask for **the full breakdown** to see per-action reasons and the rubric parts.
- **Threads:** scores the first post, then comments on whether the thread helps replies.
- **Replies and quote posts:** scored, with a note that replies get less reach.
- **Image or video posts:** judged on the text plus the visual described.
- **Multiple drafts:** one-line score each, then a rewrite of the strongest.
- **Already 9 or higher:** it says so and stops.

## Customize the voice

Rewrites are written in the author's voice, defined in [`x-review/references/swayam-voice.md`](x-review/references/swayam-voice.md). That file is specific to Swayam Dhawan (audience, tone, words to use and avoid, offers). **Replace it with your own voice guide** before using the skill for your own account.

## Repo layout

```
x-review/
  SKILL.md                      workflow, output format, hard rules
  references/
    weights.md                  X ranking weights and how to read them
    rubric.md                   the 1 to 10 scoring rubric
    rewrite-patterns.md         five rewrite structures
    swayam-voice.md             voice guide (replace with yours)
```

## Caveats

- Weights come from the public repo and can change.
- The claim that non-Premium accounts lose reach for links in the main post is reported, not confirmed in the open-source params. The skill flags body links and suggests a self-reply anyway.
- Predicted engagement is not guaranteed engagement.
