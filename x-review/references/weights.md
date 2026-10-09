# X ranking weights

Source of truth: X's open-source For You ranker, xai-org/x-algorithm (Apache 2.0). Values below are the defaults in `home-mixer/params/param.rs`, checked directly against the repo. Formula from the README: Final Score = sum of (weight x predicted probability of that action). Weights can change. If the user says they look different, re-check the repo.

## Positive weights (the ones that matter)

| Action | Param | Weight | In likes |
|---|---|---|---|
| Share via copy link | ShareViaCopyLinkWeight | 20.0 | 40 |
| Reply | ReplyWeight | 5.0 | 10 |
| Quote | QuoteWeight | 5.0 | 10 |
| Share via DM | ShareViaDmWeight | 5.0 | 10 |
| Follow author | FollowAuthorWeight | 4.0 | 8 |
| Share (generic) | ShareWeight | 2.0 | 4 |
| Repost | RetweetWeight | 1.0 | 2 |
| Like | FavoriteWeight | 0.5 | 1 |

## Small weights (ignore when writing)

| Action | Weight |
|---|---|
| Post click | 0.4 |
| Open link | 0.2 |
| Video open | 0.07 |
| Photo expand | 0.05 |
| Dwell | 0.05 |
| Quoted click | 0.05 |
| Post unexplored | 0.02 |
| Continuous dwell time | 0.004 |
| Profile click, video quality view | 0.0 |

Bookmark has no weight in param.rs. Do not count it.

## Mutual follow boost

BidirectionalFollowReplyWeightBoost = 15.0. When the viewer and author follow each other, a reply is worth 5 + 15 = 20, same as a copy link. Dwell boost for mutuals is 0.

## Negative weights

| Action | Weight |
|---|---|
| Report | -234.0 |
| Mute author | -58.8 |
| Not interested | -43.2 |
| Block author | -31.2 |
| Not dwelled (scrolled past fast) | -0.02 |

The README says a report is 468x a like, but weights scale predicted probabilities, not raw counts. So one report does not cancel 468 likes. It does mean outrage bait, spam, and fight-picking are expensive: if the model predicts even a small chance of reports, mutes, or "not interested", the score drops hard.

## Adjustments after scoring

- **Author diversity:** each extra post from the same author in a batch is multiplied by a decay (0.5), down to a floor (0.25). Posting twice in a row means the second one is discounted.
- **Out-of-network:** posts from accounts the viewer does not follow are multiplied by 0.75. Topic-based out-of-network is 0.5. So reach beyond followers is harder and needs stronger signals.
- **Low-impression boost:** authors with low impressions get a boost toward a target position. Small accounts are not locked out.
- **48 hour cutoff:** posts older than 48 hours are filtered before scoring.
- **Visibility filtering** is a separate system from ranking. It decides whether a post can be shown at all.

## How to read these correctly

1. **Predicted, not counted.** The model (Phoenix, a transformer) predicts the chance each viewer takes each action. You cannot force it by asking. You build the post so the action is likely.
2. **Copy link is 20 because it is rare.** People copy a link when the post stands alone and they know exactly who needs it. So the post must be complete, specific, and sendable with no extra context.
3. **Replies are worth 20 with mutuals.** Replying back fast to people who follow you back keeps that going.
4. **Engagement only counts on posts served in the Home timeline.** Dropping the direct link in a group chat and asking people to engage does not move ranking.
5. **Early velocity matters** in practice. Posts with no early signal fade, and nothing older than 48 hours is scored.
6. **Posting method does not matter.** Scheduled or API posts are scored the same.

## Not in the repo (reported only)

Several write-ups say non-Premium accounts that put a link in the main post see weak reach since early 2026. The open-source params do not show this, and link open is only 0.2. Treat it as reported, not confirmed. Safe advice stays the same: put the link in a self-reply.

## What each weight means for the writing

- **Copy link (20):** One complete, specific idea someone would paste to one person. Frameworks, templates, exact numbers, a line that names a situation.
- **Reply (5, or 20 with mutuals):** An easy, low-effort thing to answer. A question with a one-word answer. A take with a clear position.
- **Quote (5):** A short, sharp position people want to add their own take to.
- **DM share (5):** Makes someone think "this is for [name]". Name the audience.
- **Follow (4):** Shows a clear, repeatable value from the account.
- **Repost and like (1 and 0.5):** Ignore. Do not write for these.
- **Negatives:** Do not bait fights, do not spam, do not post near-identical posts back to back.
