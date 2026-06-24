# Community

## Community: NBA fans reacting to a live Finals game on Twitter/X (2018 NBA Finals, Game 3 — Cleveland Cavaliers vs. Golden State Warriors).

Description:

During a live NBA Finals game, fans flood Twitter with reactions in real time. These reactions are not all the same kind of talk: some are pure emotional support, some are insults aimed at players or teams, some are genuine tactical observations, and some are complaints about the referees. These distinctions matter because the same game produces very different *intents* — a community tool that surfaces tactical analysis or flags officiating disputes is far more useful than one that just measures overall sentiment. The data is a real, public dataset of ~51k tweets from this game (Kaggle), which after cleaning yields ~10k usable English fan tweets.

# Labels

1. ## Hype & Support

Definition: Tweets cheering for a team or player, expressing fandom, excitement, or admiration — positive emotion with no substantive criticism or analysis.

Examples:

"ALL HAIL THE KING! Absolutely amazing! #WhateverItTakes"
"Gooooo GS Warriors!!! #StephenCurry"
"Thank you @KingJames for being who you are."

Uncertain case:

"HOW YOU JUST GON LET @KingJames OOP ON YOU LIKE THAT?!?!! GOOD GOD!!!"

(Is this hype for LeBron, or mockery of the defender? Excitement and trash talk can overlap.)

2. ## Criticism & Trash Talk

Definition: Tweets mocking, insulting, or expressing negative judgment about a player or team — including frustration with one's own team.

Examples:

"Draymond Sucksssss"
"What's Kevin Durant's nickname again? Soft Baby Bad-actor"
"I can't stand watching the Cavs' inexplicably lazy defense down 0-2."

Uncertain case:

"Reduce the swag, FOCUS Warriors!!!"

(Frustrated criticism, or supportive coaching from a fan? Negative tone but rooting interest.)

3. ## Game/Tactical Analysis

Definition: Tweets making a substantive observation about strategy, matchups, momentum, or what a team should do — reasoning about the game, not just feeling about it.

Examples:

"Double teaming Steph, minimum fouls, minimum turnovers and connecting from 3PT range can ensure a win."
"Isn't it amazing how much better a team can look at home compared to on the road?"
"No lead is safe, so the Cavs better protect that margin before the 3rd quarter."

Uncertain case:

"Finally Kevin Love is BULLYING out there."

(Is this an analytical read on Love's play, or just excitement? Light analysis vs. hype.)

4. ## Officiating Complaint

Definition: Tweets disputing referee decisions, non-calls, or alleging unfair/biased officiating.

Examples:

"Bad no call refs!! 10:44 2nd Qtr under the Cavs basket!!!"
"Didn't know a body check was a legal screen."
"@OfficialNBARefs Really! You gentlemen have no making up to do."

Uncertain case:

"That's a travel in FIBA."

(A rules observation that could be neutral analysis, or an implied complaint that the ref missed it.)

# Mutual Exclusivity

| Label                  | Main Feature                                    |
| ---------------------- | ----------------------------------------------- |
| Hype & Support         | Positive emotion / cheering                     |
| Criticism & Trash Talk | Negative judgment / insults                     |
| Game/Tactical Analysis | Reasoning about strategy, matchups, momentum    |
| Officiating Complaint  | Grievance about referees or calls               |

# Hard Edge Cases

These are the tweet types I expect to be genuinely ambiguous between two labels, plus the tie-breaking rule I will apply consistently during annotation.

1. **Hype & Support vs. Criticism & Trash Talk** — Excitement about one team is often phrased as mockery of the other.
   - Example: _"HOW YOU GON LET @KingJames DUNK ON YOU LIKE THAT?!?!"_
   - **Tie-breaker:** Label by the tweet's *target and intent*. If the dominant act is celebrating a player/team → **Hype & Support**. If the dominant act is denigrating a player/team (even gleefully) → **Criticism & Trash Talk**. When a tweet praises one side by insulting the other, label by which is the grammatical subject / main clause.

2. **Criticism & Trash Talk vs. Game/Tactical Analysis** — A critique can be an insult or a reasoned point.
   - Example: _"The Cavs' lazy defense is why they're down 0-2."_
   - **Tie-breaker:** If the tweet gives a *reason or mechanism* (what's failing tactically and why) → **Game/Tactical Analysis**. If it's a judgment/insult with no reasoning ("they're trash") → **Criticism & Trash Talk**. Heated tone does not override the presence of real analysis.

3. **Hype & Support vs. Game/Tactical Analysis** — Praise can shade into a read on a player's impact.
   - Example: _"Finally Kevin Love is BULLYING out there."_
   - **Tie-breaker:** A tweet is **Analysis** only if it makes a claim about *how/why* the game is unfolding. A pure emotional reaction to a good play ("LOVE IS A BEAST!!!") stays **Hype & Support**.

4. **Officiating Complaint vs. Game/Tactical Analysis** — A rules observation can be neutral or a veiled grievance.
   - Example: _"That's a travel in FIBA."_
   - **Tie-breaker:** If the tweet asserts the refs *got it wrong / were unfair / missed a call* → **Officiating Complaint**. If it's a detached rules/technical observation with no grievance → **Game/Tactical Analysis**. When the complaint is the point, officiating wins.

**General annotation rule:** When two labels are still defensible after the tie-breakers, I label by the tweet's *primary communicative intent* — what the author is mainly doing (cheer, insult, analyze, dispute a call) — and I log the tweet in an "ambiguous log" with my reasoning so the decision stays consistent across the dataset.

# Data Collection Plan

**Source:** A public Kaggle dataset of tweets from the 2018 NBA Finals Game 3 (Cavaliers vs. Warriors): `archive/TweetsNBA.csv`, ~51k tweets. These are real fan reactions captured live during the game, so they reflect authentic community discourse rather than anything synthesized.

**How:** `clean_tweets.py` filters the raw dump down to a labelable pool:
- keep only English tweets (`lang == 'en'`)
- drop retweets (`RT @...`) — ~61% of the dump and pure duplicates
- strip URLs, collapse whitespace
- drop tweets that are mostly @mentions/#hashtags (low real-word content)
- length filter (30–280 chars) and exact-dedup on normalized text

This yields **~10,000 clean, unique English tweets**, from which the script samples a shuffled pool (default 500) into `raw_posts.csv` with empty `label`/`notes` columns.

**Target volume:** At least **200 labeled tweets**, aiming for a roughly balanced ~50 per class so the model sees each label enough to learn it. The natural live-tweet distribution is dominated by Hype & Support and Criticism & Trash Talk, so balancing matters.

**If a label is underrepresented after labeling the first batch** (I expect this for **Game/Tactical Analysis** and **Officiating Complaint**):
1. **Targeted search within the clean pool** — filter the 10k clean tweets for vocabulary that signals the scarce label (e.g. "refs", "no call", "travel", "foul" for officiating; "double team", "turnovers", "matchup", "rotation" for analysis) and pull more candidates.
2. **Expand the pool** — re-run `clean_tweets.py --pool 800+` to surface more candidates from the same 10k.
3. **Cap, don't fabricate** — I will not invent tweets. If a class genuinely can't reach parity, I report the final distribution and handle the imbalance via class weighting and per-class evaluation.

**Splits:** The notebook does a stratified 70/15/15 train/validation/test split. Because retweets and exact duplicates were removed during cleaning, there is **no train/test leakage** from copied text. The test set is locked and never used for tuning.

# Evaluation Metrics

Accuracy alone is misleading here for two reasons: the classes are imbalanced in the wild, and the *value* of the labels differs (analysis and officiating disputes are the rare, interesting classes). So I will use:

- **Per-class Precision, Recall, and F1** — the primary metrics. They reveal whether the model actually works on the rare, high-value labels (Game/Tactical Analysis, Officiating Complaint) instead of hiding their failure behind a high overall number.
- **Macro-averaged F1** — the single headline metric. It weights every label equally, so the model cannot "win" by only nailing the two dominant emotional classes.
- **Weighted F1** — reported alongside macro-F1 to show performance under the real class distribution.
- **Confusion matrix** — essential given the edge cases above. I specifically want to see whether the model confuses the pairs I flagged (Hype↔Criticism, Criticism↔Analysis, Hype↔Analysis, Officiating↔Analysis). The off-diagonal cells show where the label boundaries fail.

**Why these and not accuracy:** A classifier that labels everything "Hype & Support" could score high accuracy if that class dominates, while being useless for surfacing tactical analysis or officiating disputes — the very things that make the tool interesting. Macro-F1 + per-class recall measure what the tool actually needs to do.

# Definition of Success

**What makes it genuinely useful:** The tool's value is separating the *signal* (real tactical analysis, officiating disputes) from the *flood* (raw hype and trash talk). So success is defined mainly by performance on the rarer, high-value classes, not by raw accuracy.

**Targets:**
- **Macro-F1 ≥ 0.70** as the headline bar. Live tweets are short, slangy, and noisy, so I set this slightly below a clean-text task; below 0.70 the model isn't reliably separating the four intents.
- **Recall ≥ 0.65 on Game/Tactical Analysis and Officiating Complaint** — missing these defeats the purpose, since they are the rare, high-value tweets.
- **Macro-F1 should clearly beat the majority-class baseline** (always predicting the most common label). If it doesn't, the model has learned nothing useful.

**"Good enough" for deployment:** A macro-F1 in the **0.70–0.78** range with no single class F1 below **0.60**, and a confusion matrix where Hype and Criticism aren't being systematically merged. At that point the tool is a useful *assistant* — it pre-sorts the live feed and surfaces analysis/officiating tweets, with a human reviewing borderline/low-confidence cases rather than relying on it to act autonomously.

**What I would not accept:** High accuracy driven by the dominant emotional classes while Analysis or Officiating sits below ~0.5 F1, or systematic confusion between Hype & Support and Criticism & Trash Talk (which would make the sentiment split meaningless).

# AI Tool Plan

This is an annotation-and-evaluation project, not an implementation project, so there is no model code for AI to generate. AI tools help in three specific places.

## 1. Label Stress-Testing (before annotation)

**What I'll do:** Give the AI my four label definitions, examples, uncertain cases, and the Hard Edge Cases section, then ask it to generate **5–10 tweets that deliberately sit on the boundary between two labels** (e.g., hype phrased as an insult, or a critique that contains real analysis).

**How I'll act on it:** I classify each generated tweet using only my written definitions and tie-breakers. If any is genuinely unclassifiable, that exposes a gap — **I tighten the definition/tie-breaker now, before annotating 200 real tweets.** I re-run until the boundaries hold. Generated tweets are kept separate and never enter the real dataset.

**Tool:** Claude (Opus 4.8).

## 2. Annotation Assistance (during labeling)

**Decision: Yes, I will use an LLM to pre-label, with full human review.** Hand-labeling hundreds of tweets cold is slow and drifts; pre-labeling gives me a first pass to react to.

**Process:**
- The LLM gets my label definitions + tie-breakers and proposes one label per tweet from `raw_posts.csv`.
- **I review every pre-labeled tweet myself** — the LLM's label is a suggestion, never final. The human label is what enters the dataset.
- **Tracking for disclosure:** each row carries a `pre_labeled` flag and an `llm_label` column alongside the final `label`. This lets me (a) report exactly which tweets were AI-assisted in my AI usage section, and (b) measure LLM–human agreement as a signal of which labels are ambiguous.
- I will **not** pre-label the test set — it is hand-annotated from scratch to avoid biasing ground truth toward the model's tendencies.

## 3. Failure Analysis (after evaluation)

**What I'll do:** Export the misclassified test tweets (text, true label, predicted label, confidence) and ask the AI to **cluster the errors into patterns** before I write up the evaluation.

**What I'll look for:**
- Which label pairs dominate the confusion matrix (do they match my predicted edge cases — Hype↔Criticism, Officiating↔Analysis?).
- Systematic triggers: sarcasm, ALL-CAPS hype, heavy slang/abbreviations, hashtag-laden tweets, tweets mixing two intents.
- Whether errors concentrate in one direction (e.g. Officiating Complaint under-predicted), flagging a recall problem on a high-priority label.

**How I'll verify the patterns myself:** AI-proposed patterns are hypotheses. For each claimed pattern I pull the actual misclassified tweets behind it and read them, then check the count against the confusion matrix. A pattern only enters the write-up if I can point to the specific tweets supporting it and the numbers hold up — I won't repeat an AI explanation I haven't confirmed.
