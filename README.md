# TakeMeter — r/LiverpoolFC Discourse Classifier

**AI201 Project 3 | Guillaume Girishya | Member ID 155499**

---

## Community Choice

**r/LiverpoolFC** (645,000+ members) is one of the largest football club subreddits. It was chosen because its posts fall naturally into distinct discourse modes: raw emotional reactions to goals and results, bold opinions about players and managers, evidence-based tactical arguments, and transfer news. These four types are genuinely different in tone, structure, and intent — making it a strong candidate for a text classifier. The community is also highly active, providing abundant labeled data from a single consistent source.

---

## Label Taxonomy

| Label | Definition |
|---|---|
| `analysis` | Structured argument backed by stats, tactical observation, historical comparison, or specific evidence. The writer is reasoning from evidence, not just reacting or asserting. |
| `hot_take` | Bold, confident opinion stated without real supporting evidence. Asserts rather than argues. |
| `matchday_reaction` | Immediate emotional response tied to a specific match or moment. Little to no argument — expressing a feeling in real time. |
| `transfer_rumor` | Primarily about player movement — signings, departures, contract news, or speculation about deals. Focus is on the transfer itself, not tactical analysis. |

**Examples per label:**

**`analysis`**
- *"Slot switched Salah to more of an inside forward role not tracking back, switched formation to a double pivot with a 10 instead of 4-3-3 which allowed Gravenberch to flourish, and started using Diaz as a false 9 — those are big changes that directly explain last season's results."*
- *"There are interviews where Trent has said Klopp literally wanted him to hit those long balls to create a counterpress situation after the cross turnover, which effectively creates defensive disorganisation if we win the ball back."*

**`hot_take`**
- *"Lowest 38-game points total since we fired Brendan. Writing is on the wall. Get him out."*
- *"FSG aren't the great owners people make them out to be — they were carried by Klopp and made to look better than they actually are."*

**`matchday_reaction`**
- *"Newcastle [2] - [3] Liverpool - R. Ngumoha 90+9'"*
- *"They won the f***ing lot."*

**`transfer_rumor`**
- *"[Romano] EXCLUSIVE: Florian Wirtz to Liverpool HERE WE GO! Liverpool verbally agree deal in principle with Bayer Leverkusen for package reaching €150m add-ons included."*
- *"[Joyce] Liverpool and Newcastle United agree Alexander Isak deal at £125 million. Player expected to sign six-year contract."*

**Edge case decision rules:**
- Transfer topic + opinion with no rumor attached (e.g. "we should sign X") → `hot_take`
- Transfer topic + stat-based argument for why the player fits the system → `analysis`
- Stat used as decoration for a pre-formed opinion → `hot_take`, not `analysis`

---

## Data Collection

**Sources:**
- Top posts from r/LiverpoolFC (top/year, 100 posts) — primarily yielded `matchday_reaction` due to Diogo Jota tribute posts dominating the top of the year
- Comment thread on a Klopp vs Slot tactical comparison post (post ID: 1rvtt9u) — yielded `analysis` and `hot_take`
- Reddit search for "transfer" — yielded `transfer_rumor`
- Unpopular Opinions megathread (post ID: 1slwegj) — yielded `hot_take` and `analysis`

**Labeling process:** All 220 examples were labeled manually. Each post was read in full and assigned a single label based on the taxonomy above. When a post touched multiple categories, the primary intent determined the label (e.g. a transfer post that also contained tactical opinion was labeled `transfer_rumor` if the transfer was the main subject).

**Label distribution:**

| Label | Count | % |
|---|---|---|
| `matchday_reaction` | 82 | 37.3% |
| `hot_take` | 66 | 30.0% |
| `transfer_rumor` | 44 | 20.0% |
| `analysis` | 28 | 12.7% |

The dataset skews toward `matchday_reaction` because the top posts from the past year were dominated by Diogo Jota tribute posts, which are emotional by nature. `analysis` is the smallest class because genuine evidence-based arguments are rarer in the wild than reactions or hot takes.

**3 difficult-to-label examples:**

1. *"Stats since April last year, last 38 games. The only reason we won last year is because they were playing as a team. Now they are playing as individuals and poorly coached."*
   → **Decision: `analysis`**. The post references specific stats and makes a causal argument from them. Although the conclusion ("poorly coached") sounds like a hot take, the writer is reasoning from evidence rather than asserting without support.

2. *"Klopp watching this transfer window"* (meme reference)
   → **Decision: `hot_take`**. No transfer rumor is attached — it's an opinion about FSG's transfer activity expressed through a cultural reference. No evidence, just assertion.

3. *"[Official] Liverpool FC can confirm Arne Slot is to depart"*
   → **Decision: `transfer_rumor`**. Although this is about a manager departure rather than a player transfer, the post is fundamentally about personnel movement at the club, which fits the spirit of the label.

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot)

**Prompt used:**

```
You are classifying posts and comments from r/LiverpoolFC, a subreddit for Liverpool Football Club fans.
Assign each post to exactly one of the following categories.

analysis: A structured argument backed by stats, tactical observation, historical comparison, or specific evidence. The writer is reasoning from evidence, not just reacting or asserting.
Example: "Slot switched Salah to more of an inside forward role..."

hot_take: A bold, confident opinion stated without real supporting evidence. The post asserts a strong claim rather than arguing for it.
Example: "Mac Allister is overrated, never shows up in big games."

matchday_reaction: An immediate emotional response tied to a specific match, goal, or moment. Little to no argument.
Example: "ABSOLUTELY INCREDIBLE. WHAT A GOAL. I love this team so much."

transfer_rumor: A post primarily about player movement — signings, departures, contract news, or speculation.
Example: "Fabrizio Romano: Liverpool in advanced talks with Bayer Leverkusen for Wirtz, here we go soon."

Respond with ONLY the label name. Do not explain your reasoning.

Valid labels:
analysis
hot_take
matchday_reaction
transfer_rumor
```

**How results were collected:** Each post in the test set was sent to the Groq API individually with the system prompt above. The model's response was stripped and matched against the four valid label strings. All 33 test posts returned parseable responses (100% parse rate).

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (66M parameters, uncased English)

**Training setup:**
- Train/val/test split: 70% / 15% / 15% (154 train, 33 val, 33 test)
- Tokenizer: DistilBERT tokenizer, max length 256 tokens
- Labels encoded from the label map: `matchday_reaction=0`, `hot_take=1`, `analysis=2`, `transfer_rumor=3`

**Hyperparameters:**

| Parameter | Value | Reasoning |
|---|---|---|
| `num_train_epochs` | 3 | Standard starting point for small datasets; more epochs risk overfitting on 220 examples |
| `learning_rate` | 2e-5 | Standard for fine-tuning BERT-family models; lower = more stable |
| `per_device_train_batch_size` | 16 | Fits T4 GPU comfortably without OOM errors |
| `weight_decay` | 0.01 | Light regularization to reduce overfitting |
| `warmup_steps` | 50 | Gradual learning rate warmup to stabilize early training |
| `load_best_model_at_end` | True | Saves the checkpoint with best validation accuracy |

**Key hyperparameter decision:** `num_train_epochs=3` was chosen deliberately over a higher value. With only 154 training examples and significant class imbalance, more epochs risk overfitting to majority classes. The model was evaluated after each epoch and the best checkpoint was kept.

---

## Evaluation Report

### Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq llama-3.3-70b) | **84.8%** |
| Fine-tuned DistilBERT | **78.8%** |

Fine-tuning regression: **0.061** (baseline still wins, but fine-tuned model is much more competitive than the first run)

### Per-Class Metrics

**Baseline (Groq):**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| matchday_reaction | 0.92 | 0.92 | 0.92 | 13 |
| hot_take | 1.00 | 0.60 | 0.75 | 10 |
| analysis | 0.67 | 1.00 | 0.80 | 4 |
| transfer_rumor | 0.75 | 1.00 | 0.86 | 6 |
| **macro avg** | **0.83** | **0.88** | **0.83** | 33 |

**Fine-tuned DistilBERT:**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| matchday_reaction | 0.76 | 1.00 | 0.87 | 13 |
| hot_take | 0.73 | 0.80 | 0.76 | 10 |
| analysis | 0.00 | 0.00 | 0.00 | 4 |
| transfer_rumor | 1.00 | 0.83 | 0.91 | 6 |
| **macro avg** | **0.62** | **0.66** | **0.63** | 33 |

### Confusion Matrix

**Fine-tuned model (test set):**

| | Pred: matchday_reaction | Pred: hot_take | Pred: analysis | Pred: transfer_rumor |
|---|---|---|---|---|
| **True: matchday_reaction** | 13 | 0 | 0 | 0 |
| **True: hot_take** | 2 | 8 | 0 | 0 |
| **True: analysis** | 1 | 3 | 0 | 0 |
| **True: transfer_rumor** | 1 | 0 | 0 | 5 |

### 3 Wrong Predictions Analyzed

**#1 — True: `hot_take` → Predicted: `matchday_reaction` (confidence: 0.27)**
> *"Despite Isak scoring for Liverpool Newcastle won't sell"*
> This references a specific match event (Isak scoring) which the model associated with matchday reaction. But the actual claim — that Newcastle won't sell regardless — is an unsupported assertion about transfer policy. The model was misled by the match reference in the first half of the sentence and never reached the hot take in the second half.

**#2 — True: `analysis` → Predicted: `hot_take` (confidence: 0.30)**
> *"I don't think us playing midweek fixtures counts for much. We have a team that's used to a congested schedule. We actually have a bigger squad than Man Utd."*
> The model confused analysis with hot_take here. The post does make claims without quoting specific stats, but the writer is constructing a reasoned argument using squad context and scheduling evidence. The model never learned to distinguish between "assertion backed by reasoning" and "assertion with no backing" — both can look similar at the surface level.

**#3 — True: `analysis` → Predicted: `hot_take` (confidence: 0.29)**
> *"As the famous saying goes why fix something that isn't broken. It was working fine as long as he didn't change the tactics much. Once he started imprinting his tactics on the team we started to regress..."*
> The casual register ("as the famous saying goes") and opinionated conclusion made the model think this was a hot take. But the post makes a specific causal argument about tactical change causing regression — that's analysis. The model struggled with the `analysis` class throughout, getting an F1 of 0.00 for it.

### Sample Classifications Table

| Post (truncated) | True Label | Predicted Label | Confidence | Correct? |
|---|---|---|---|---|
| "Kelleher saved Cristiano Ronaldo penalty" | matchday_reaction | matchday_reaction | 0.30 | ✅ |
| "No managers could've survived this cursed season." | hot_take | hot_take | 0.28 | ✅ |
| "[LFC] Ryan Gravenberch signed a new long-term contract" | transfer_rumor | transfer_rumor | 0.29 | ✅ |
| "Despite Isak scoring for Liverpool Newcastle won't sell" | hot_take | matchday_reaction | 0.27 | ❌ |
| "I don't think us playing midweek fixtures counts for much. We have a bigger squad than Man Utd." | analysis | hot_take | 0.30 | ❌ |

**Correct prediction explained:** *"Kelleher saved Cristiano Ronaldo penalty"* was correctly labeled `matchday_reaction`. The post names a specific player, describes a single match event, and has the short declarative structure typical of live match updates. These surface features appear consistently across `matchday_reaction` examples in the training data, making it one of the model's most reliable patterns.

---

## Reflection

**What the model learned vs. what I intended:**

I intended the model to learn the semantic difference between four types of discourse: evidence-based reasoning, unsupported assertion, emotional reaction, and factual news. What it actually learned was a reasonable approximation for three of the four: it identified `matchday_reaction` well (F1 0.87), `transfer_rumor` well (F1 0.91), and `hot_take` decently (F1 0.76). The failure was entirely on `analysis` (F1 0.00) — the model consistently confused it with `hot_take`.

The core issue is that `analysis` and `hot_take` are the hardest to distinguish from surface features alone. Both can be written in opinionated, confident language. The difference is whether the writer reasons from evidence — a semantic distinction that requires understanding argument structure, not just word patterns. DistilBERT at 66M parameters, trained on only 20 analysis examples, never developed that capability.

**What I would do differently:**
1. Collect at least 60 `analysis` examples before training — 20 in the training split is not enough
2. Use a weighted loss function to penalize the model more for missing minority classes
3. Evaluate per-class F1 during training, not just overall accuracy — the model could achieve 79% overall while completely ignoring `analysis`

---

## Spec Reflection

**One way the spec helped:** Defining edge case decision rules before data collection was essential. The rule "transfer topic + stat-based argument → `analysis`, not `transfer_rumor`" came up repeatedly during labeling and having it written down kept the labels consistent. Without it, those borderline posts would have been labeled inconsistently.

**One way implementation diverged from the spec:** The spec assumed roughly equal label distribution across classes. In practice, the top posts from r/LiverpoolFC for the past year were dominated by Diogo Jota tribute posts (he passed away during the season), which are all `matchday_reaction`. This was unforeseeable during planning and meant the dataset ended up more imbalanced than intended — particularly hurting the `analysis` class which ended up with only 28 examples total (20 in training).

---

## AI Usage

This project used Claude (Anthropic) as a collaborative tool throughout. Two specific instances:

**1. Dataset labeling assistance**
I pasted raw Reddit JSON into Claude and asked it to extract post text and suggest labels based on the taxonomy I had defined. I reviewed every suggested label individually and overrode several , for example, Claude initially labeled *"Stats since April last year, last 38 games..."* as `hot_take` because the conclusion was assertive, but I changed it to `analysis` because the post was explicitly reasoning from a statistic. The final labels reflect my judgment, with Claude handling the mechanical extraction work.

**2. README and planning.md drafting**
I directed Claude to draft the planning.md and README structure based on the rubric requirements and my notes. I then reviewed each section, corrected factual details (exact post counts, specific wrong predictions, real metric numbers from the notebook), and revised the failure analysis based on what I actually observed in the output. The reflection sections were written collaboratively, I described what I observed, Claude structured the explanation, and I revised for accuracy.

All annotation decisions were made by me. Claude was used as a writing and extraction tool, not as the final decision-maker on labels.

---

## Files

| File | Description |
|---|---|
| `takemeter_dabase.csv` | 220 labeled examples (text, label) |
| `planning.md` | Label design, data collection plan, evaluation criteria |
| `confusion_matrix.png` | Fine-tuned model confusion matrix on test set |
| `evaluation_results.json` | Baseline and fine-tuned accuracy scores |
| `Takemeter-Liverpool.ipynb` | Colab notebook with baseline and fine-tuning code |
