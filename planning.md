# TakeMeter — Planning Document
**Community:** r/LiverpoolFC
**Project:** AI201 Project 3

---

## 1. Community
r/LiverpoolFC is one of the largest football club subreddits, with highly 
active discourse ranging from tactical breakdowns to emotional matchday 
reactions to transfer speculation. The community is a strong fit for 
classification because post types are genuinely varied — the same topic 
(e.g. a player's form) can produce an evidence-backed argument, a bold 
unsupported opinion, or a pure emotional reaction depending on the poster.

---

## 2. Label Taxonomy

### `analysis`
The post makes a structured argument backed by stats, tactical observation,
historical comparison, or specific evidence. The writer is reasoning, not
just reacting.
- "Salah's pressing numbers this season are down 15% — Slot is using him
more as a finisher than a presser compared to Klopp's system"
- "Our xG against low-block defenses has been poor all season, here's why
the 4-3-3 struggles..."

### `hot_take`
A bold, confident opinion stated without real supporting evidence. The claim
might be right, but the post asserts rather than argues.
- "Mac Allister is overrated, never shows up in big games"
- "We should sell Trent, he's been a liability for 2 years"

### `matchday_reaction`
An immediate emotional response tied to a specific match or moment. Little
to no argument — the post is expressing a feeling in real time.
- "ABSOLUTELY INCREDIBLE. WHAT A GOAL"
- "That referee is an absolute joke, same every time we play away"

### `transfer_rumor`
A post primarily about player movement — signings, departures, contract
news, or speculation about incoming/outgoing players. The focus is on the
transfer itself, not tactical analysis of whether it's a good idea.
- "Fabrizio Romano: Liverpool in talks with Bayer Leverkusen for Wirtz"
- "Hearing we're close to signing a new left back, fee around £40m"

---

## 3. Hard Edge Cases

**Transfer + opinion:** "We should sign Gyokeres, he'd be perfect for our 
system"
- Could be `hot_take` or `transfer_rumor`
- Decision rule: if the post is reporting/speculating about a specific deal,
label it `transfer_rumor`. If it's arguing we should sign someone without 
a rumor attached, label it `hot_take`.

**Transfer + analysis:** "Wirtz's heatmap and pressing stats show he'd slot 
perfectly into our 4-3-3"
- Could be `analysis` or `transfer_rumor`
- Decision rule: if the post uses evidence to argue the case for a transfer,
label it `analysis`. The transfer is the topic but reasoning is the mode.

**Stat-backed hot take:** "Trent has been awful, his defensive stats are the
worst in the league"
- Could be `hot_take` or `analysis`
- Decision rule: if the stat is specific and verifiable and the post reasons
from it, label it `analysis`. If the stat feels cherry-picked to back up an
already-formed opinion, label it `hot_take`.

---

## 4. Data Collection Plan
- Source: r/LiverpoolFC (public posts and comments)
- Target: 200 examples, aiming for ~50 per label
- Method: manual copy-paste into a CSV file
- If a label is underrepresented after 150 examples, I will specifically
search for posts of that type (e.g. search "xG" or "stats" for analysis
posts, search match threads for matchday_reaction)

---

## 5. Evaluation Metrics
- **Accuracy:** overall fraction correct — useful but not enough alone
- **Per-class F1:** harmonic mean of precision and recall per label —
important because some labels may be harder to learn than others
- **Confusion matrix:** shows which label pairs the model confuses most,
giving actionable insight into where the boundary is unclear
- Accuracy alone is insufficient because a model that always predicts
`hot_take` could score well if that label dominates the dataset

---

## 6. Definition of Success
- Overall accuracy above 70% on the test set
- No single label with F1 below 0.50
- Fine-tuned model meaningfully outperforms the zero-shot Groq baseline
- The confusion matrix shows no single off-diagonal cell dominating errors

---

## 7. AI Tool Plan
- **Label stress-testing:** I will probably use Claude to generate 10 boundary posts between `hot_take` and `analysis` to pressure-test label definitions before
annotating
- **Annotation assistance:** I may use an LLM to pre-label batches of posts,
then review and correct every label myself before saving
- **Failure analysis:** after training, paste misclassified examples into
Claude and ask it to identify error patterns, then verify manually