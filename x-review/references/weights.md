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

## Small weights

| Action | Param | Weight |
|---|---|---|
| Click dwell time (time spent after opening the post) | ContClickDwellTimeWeight | 0.4 |
| Post click | ClickWeight | 0.3 |
| Open link | OpenLinkWeight | 0.2 |
| Video open | VideoOpenWeight | 0.07 |
| Photo expand | PhotoExpandWeight | 0.05 |
| Dwell | DwellWeight | 0.05 |
| Quoted click | QuotedClickWeight | 0.05 |
| Post unexplored | PostUnexploredWeight | 0.015 (0.02 in vm-ranker) |
| Continuous dwell time | ContDwellTimeWeight | 0.004 |
| Profile click, video quality view (watch) | ProfileClickWeight, VqvWeight | 0.0 |

Individually small, but click (0.3) + click dwell time (0.4) together are worth more than a like (0.5). A post that earns a "Show more" tap and a real read is quietly scoring. Video watch time itself scores 0, so judge video posts on whether they earn shares and replies, not views.

## Mutual follow boost

BidirectionalFollowReplyWeightBoost = 15.0. When the viewer and the author follow each other, the predicted chance that the viewer replies to the author's original post is weighted 5 + 15 = 20, the same as a copy link. It applies to original posts, not to your replies. Dwell boost for mutuals is 0.

What this means for writing: the people most likely to reply are your mutuals. Ask questions your mutuals (peers in your niche) can answer from experience. Growing real mutual follows in your niche raises the value of every future post.

## Negative weights

| Action | Param | Weight |
|---|---|---|
| Report | ReportWeight | -234.0 |
| Mute author | MuteAuthorWeight | -58.8 |
| Not interested | NotInterestedWeight | -47.52 |
| Block author | BlockAuthorWeight | -31.2 |
| Not dwelled (scrolled past fast) | NotDwelledWeight | -0.02 |

The README says a report is 468x a like, but weights scale predicted probabilities, not raw counts. So one report does not cancel 468 likes. It does mean outrage bait, spam, and fight-picking are expensive: if the model predicts even a small chance of reports, mutes, or "not interested", the score drops hard.

## Adjustments after scoring

- **Author diversity:** each extra post from the same author in one feed batch is multiplied by 0.5, down to a floor of 0.25. Then a DPP diversity rerank pushes similar-looking posts apart. Space posts out and do not post near-duplicates.
- **Out-of-network:** posts shown to people who do not follow you are multiplied by 0.75 (topic-based out-of-network 0.5). Reach beyond followers needs stronger predicted shares and replies.
- **Your replies and reposts:** shown to your followers at x0.75 (in-network rescore). Shown to non-followers, they are filtered out of For You entirely. So replies and reposts are not a reach strategy. Original posts are.
- **Cold-start boost:** under 200 impressions, under 50k followers, post under 2h old: lifted toward slot ~15.
- **48 hour cutoff:** posts older than 48 hours are dropped before scoring.
- **Visibility filtering** is a separate system from ranking. It decides whether a post can be shown at all.

## How to read these correctly

1. **Predicted, not counted.** The model (Phoenix, a transformer) predicts the chance each viewer takes each action. You cannot force it by asking. You build the post so the action is likely.
2. **Copy link is 20 because it is rare.** People copy a link when the post stands alone and they know exactly who needs it. So the post must be complete, specific, and sendable with no extra context.
3. **Replies are worth 20 from mutuals.** The ranker scores the chance a viewer replies to your post. There is no "author replied back" weight in the 2026 code (the 75x author-reply value was 2023). Replying to people is still good for relationships and creates more mutuals, but do not tell the user it is a ranking multiplier.
4. **Engagement only counts on posts served in the Home timeline.** Dropping the direct link in a group chat and asking people to engage does not move ranking.
5. **First 2 hours matter for small accounts.** Cold-start boost (author_cold_start.rs): if the author has under 200 impressions, fewer than 50,000 followers, and the post is under 2 hours old, the post can be lifted toward feed slot ~15. After 48 hours a post is not scored at all. "Early velocity" beyond that is reported, not in the code.
6. **Posting method does not matter.** Scheduled or API posts are scored the same.

## Links and Premium

There is no link penalty and no Premium or verified boost anywhere in the 2026 For You ranking code. (The 2023 code had x4 / x2 Premium boosts; they are gone.) Open link is +0.2, so a link is not penalized by the ranker. The real cost of a body link is indirect: people leave the app instead of replying or sharing. X marketing only claims Premium affects reply-thread ordering. Advice: a link in the body is fine if the post stands alone without it; put it in a self-reply when the post is mainly a pointer to the link.

## What each weight means for the writing

- **Copy link (20):** One complete, specific idea someone would paste to one person. Frameworks, templates, exact numbers, a line that names a situation.
- **Reply (5, or 20 with mutuals):** An easy, low-effort thing to answer. A question with a one-word answer. A take with a clear position.
- **Quote (5):** A short, sharp position people want to add their own take to.
- **DM share (5):** Makes someone think "this is for [name]". Name the audience.
- **Follow (4):** Shows a clear, repeatable value from the account.
- **Repost and like (1 and 0.5):** Ignore. Do not write for these.
- **Negatives:** Do not bait fights, do not spam, do not post near-identical posts back to back.
- **Click + click dwell (0.3 + 0.4):** A strong first line that earns "Show more" and a full read. Worth more than a like combined.
- **Not interested (-47.52):** The biggest everyday risk. Off-niche posts, vague hype, and repeated pitches trigger it more than outrage does.
