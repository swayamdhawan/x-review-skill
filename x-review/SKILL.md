---
name: x-review
description: Scores an X (Twitter) post from 1 to 10 against X's open-source ranking weights, names which actions it will earn (copy link, reply, quote, DM share, follow), then rewrites it until it scores 9 or higher in Swayam's voice. Use whenever the user types /x-review, or says "review my X post", "score this tweet", "will this post get reach", "rewrite this for X", "make this post follow the algorithm", or pastes a draft post for X and wants it checked before publishing. Trigger even if they only paste a draft and say "thoughts?" in an X context.
---

# x-review

Scores a draft X post against the real ranking weights, then rewrites it so it earns the actions that move reach. Based on the prompt Noah Kagan (AppSumo) shared: score the post using X's weights, say which actions it will earn, rewrite it to earn a share or a reply.

## Why this works

X's For You feed scores each post as a weighted sum of predicted actions. It predicts what people will do. It does not count what already happened. Likes are almost worthless. Copy link shares and replies from mutuals are worth the most. So a good post is built around one question: what will make someone copy this link and send it to one person, or reply?

Weights are verified against the repo (xai-org/x-algorithm, home-mixer/params/param.rs). Read `references/weights.md` before scoring.

## Workflow

1. **Get the draft.** If the user ran /x-review with no post, ask for it in one line. If they pasted a post, go straight to scoring. If they attached an image of a post, read the text from it.
2. **Read the weights.** Open `references/weights.md`. Use the numbers there, not memory.
3. **Score the draft.** Use the rubric in `references/rubric.md`. Score 1 to 10. Be strict. Most first drafts land at 4 to 6.
4. **Name the actions it will earn.** Only list the actions it is Likely or Possible to earn, with the weight in brackets, ordered by weight. Weights: copy link 20, reply 5 (20 with mutuals), quote 5, DM share 5, follow 4, repost 1, like 0.5. Ignore repost and like unless they are the only thing it earns.
5. **Name the one biggest leak.** The single thing costing the most points. Not a list of ten.
6. **Rewrite.** Use `references/rewrite-patterns.md` for the structure and `references/swayam-voice.md` for the voice. Give one main rewrite. Add one alternate only if the post could go two genuinely different ways (for example a reply-bait version and a save-and-send version).
7. **Re-score the rewrite** with the same rubric. If it is under 9, fix it and re-score once more. Show the final score only, not every loop.
8. **Add the posting note** (see Output format).

## Output format

Short. No intro, no recap. Use this order:

**Score: X/10**

**Likely to earn:** copy link (20), replies (5). Possible: quotes (5). [one line on the words in the post that drive it]

**One fix:** the single biggest leak, one sentence. If the post has a bait CTA, a link in the body, or outrage bait, say that here.

**Rewrite (Y/10)**
[the post, ready to paste, in a plain block]

**Why it scores higher:** one or two lines. Which action it now earns and what changed.

**Posting note:** one line. Examples: "Put the link in a self-reply." "Reply to the first 5 repliers within 30 minutes." "Do not post another one within a few minutes."

If the user asks for the full breakdown, add the per-action reasons (Likely / Possible / Unlikely) and the rubric parts.

## Hard rules

- No em dashes anywhere in the output. Use periods, commas, colons.
- Never invent numbers, client results, or stories. If the rewrite needs a specific the user did not give, put it in [brackets] and tell them to fill it in with their real number.
- Do not claim a post will get a certain number of views. Scores are a prediction of what the post is built to earn, not a guarantee.
- Do not recommend begging for engagement. "Like and RT" and "comment YES" reads as bait and does not earn the heavy actions.
- Do not use hype words or guru tone. The voice is calm, direct, specific.
- If the draft is already 9 or higher, say so in one line, give one small tweak at most, and stop. Do not rewrite for the sake of rewriting.
- If the draft has a link in the post body, flag it. Link open is only 0.2 in the repo, and reports say links in the main post hurt reach for non-Premium accounts. That part is reported, not confirmed. Suggest moving it to a self-reply.
- Do not touch the user's facts. Change structure, hook, and ask. Keep their claim.

## Edge cases

- **Thread:** Score the first post only. Then say in one line whether the thread structure helps replies. Offer to score each post on request.
- **Reply or quote post (not original):** Score it, but note that replies get less reach than original posts. Focus the rewrite on being the reply worth quoting.
- **Image or video post:** Judge the text plus the visual described. A screenshot that someone would send to a friend is a copy link and DM share signal.
- **Very short post (under 12 words):** Usually low on reply and share. Ask whether it has a hook and a point. Rewrite to add one concrete thing.
- **Multiple drafts pasted:** Score each in one line, then rewrite only the strongest.
