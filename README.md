# TakeMeter — r/LiverpoolFC Discourse Classifier

**AI201 Project 3 | Guillaume Girishya | Member ID 155499**

---
## Community Choice

**r/LiverpoolFC** (645,000+ members) is one of the largest football club subreddits. I chose it because I am a liverpool fan and, its posts fall naturally into distinct discourse modes: raw emotional reactions to goals and results, bold opinions about players and managers, evidence-based tactical arguments, and transfer news. These four types are genuinely different in tone, structure, and intent, making it a strong candidate for a text classifier. The community is also highly active, providing abundant labeled data from a single consistent source.

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
- *"They won the whole lot."*

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

| Label               |Count|    %  |
|---------------------|---- |-------|
| `matchday_reaction` | 82  | 37.3% |
| `hot_take`          | 66  | 30.0% |
| `transfer_rumor`    | 44  | 20.0% |
| `analysis`          | 28  | 12.7% |

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
- Tokenizer: DistilBERT tokenizer, max length 128 tokens
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

**Key hyperparameter decision:** `num_train_epochs=3` was chosen deliberately over a higher value. With only 154 training examples and significant class imbalance, more epochs would have caused the model to overfit further to `matchday_reaction` rather than learning the minority classes. Even at 3 epochs the model collapsed — more epochs would have made this worse.

---
## Evaluation Report

### Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq llama-3.3-70b) | **84.8%** |
| Fine-tuned DistilBERT | **45.5%** |

Fine-tuning regression: **0.394**

### Per-Class Metrics

**Baseline (Groq):**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| matchday_reaction | 0.91 | 0.83 | 0.87 | 12 |
| hot_take | 1.00 | 0.70 | 0.82 | 10 |
| analysis | 0.80 | 1.00 | 0.89 | 4 |
| transfer_rumor | 0.70 | 1.00 | 0.82 | 7 |
| **macro avg** | **0.85** | **0.88** | **0.85** | 33 |

**Fine-tuned DistilBERT:**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| matchday_reaction | 0.43 | 1.00 | 0.60 | 12 |
| hot_take | 0.60 | 0.30 | 0.40 | 10 |
| analysis | 0.00 | 0.00 | 0.00 | 4 |
| transfer_rumor | 0.00 | 0.00 | 0.00 | 7 |
| **macro avg** | **0.26** | **0.33** | **0.25** | 33 |

### Confusion Matrix

**Fine-tuned model (test set):**

| | Pred: matchday_reaction | Pred: hot_take | Pred: analysis | Pred: transfer_rumor |
|---|---|---|---|---|
| **True: matchday_reaction** | 12 | 0 | 0 | 0 |
| **True: hot_take** | 7 | 3 | 0 | 0 |
| **True: analysis** | 2 | 2 | 0 | 0 |
| **True: transfer_rumor** | 7 | 0 | 0 | 0 |

### 3 Wrong Predictions Analyzed

**#1 — True: `transfer_rumor` → Predicted: `matchday_reaction` (confidence: 0.32)**
> *"Liverpool FC complete signing of Thiago Alcantara"*
> This is a short announcement-style post with no transfer-specific vocabulary (no "fee", "contract", "Romano", "here we go"). The fine-tuned model never learned the `transfer_rumor` pattern — it predicted 0 correct transfer rumors across the entire test set. The post's brevity and declarative tone were likely associated with matchday announcements in the training data.

**#2 — True: `analysis` → Predicted: `matchday_reaction` (confidence: 0.29)**
> *"There are interviews where Trent has said Klopp literally wanted him to hit those long balls to create a counterpress situation after the cross turnover which effectively creates defensive disorganisation if we win the ball back."*
> The model had only 28 analysis examples in training (13% of data) — not enough to learn what careful tactical reasoning looks like. The low confidence score (0.29) confirms the model was essentially guessing. It defaulted to the majority class.

**#3 — True: `hot_take` → Predicted: `matchday_reaction` (confidence: 0.31)**
> *"No managers could've survived this cursed season."*
> This post has emotional urgency and implicitly references the current season, which the model confused for a matchday reaction. The distinction between "reacting emotionally to a moment" and "making an unsupported claim about the season" is subtle — and the model never learned it because it collapsed to predicting `matchday_reaction` for anything that didn't look like a `hot_take` by surface features.

### Sample Classifications Table

| Post (truncated) | True Label | Predicted Label | Confidence | Correct? |
|-------------------|---|---|---|---|
| "Newcastle [2] - [3] Liverpool - R. Ngumoha 90+9'" | matchday_reaction | matchday_reaction | 0.32 | ✅ |
| "No managers could've survived this cursed season." | hot_take | matchday_reaction | 0.31 | ❌ |
| "Liverpool FC complete signing of Thiago Alcantara" | transfer_rumor | matchday_reaction | 0.32 | ❌ |
| "Slot will stay and we'll have a much better run under him next season" | hot_take | hot_take | 0.31 | ✅ |
| "There are interviews where Trent has said Klopp literally wanted him to hit long balls to create a counterpress situation..." | analysis | matchday_reaction | 0.29 | ❌ |

**Correct prediction explained:** *"Newcastle [2] - [3] Liverpool - R. Ngumoha 90+9'"* was correctly labeled `matchday_reaction`. This post follows a strict scoreline format that appears frequently in the training data. The bracketed score notation, player name, and minute marker are strong surface-level features the model learned to associate with live match updates — the one pattern it reliably got right.

---


## Reflection

**What the model learned vs. what I intended:**

I intended the model to learn the semantic difference between four types of discourse: evidence-based reasoning, unsupported assertion, emotional reaction, and factual news. What it actually learned was a much simpler heuristic: if the text looks like a live match update or short exclamation, predict `matchday_reaction`; if it looks like a longer opinionated sentence, try `hot_take`; otherwise default to `matchday_reaction`. It never learned `analysis` or `transfer_rumor` at all — both got F1 scores of 0.00.

The core problem is that DistilBERT needed more labeled examples than I gave it, especially for the minority classes. With 28 analysis examples and 44 transfer rumors, the model could achieve 37% accuracy by guessing `matchday_reaction` every time — which was good enough to minimize its training loss without learning the harder distinctions.

**What I would do differently:**
1. Balance the dataset before training — oversample `analysis` and `transfer_rumor` to at least 60 examples each, or use a weighted loss function
2. Collect more `analysis` examples specifically — 28 is insufficient for a 66M parameter model to learn a nuanced category
3. Evaluate per-class F1 during training, not just accuracy — a model ignoring two classes entirely can still report 45% overall accuracy, which masks the failure

---

## Spec Reflection

**One way the spec helped:** Defining edge case decision rules before data collection was essential. The rule "transfer topic + stat-based argument → `analysis`, not `transfer_rumor`" came up repeatedly during labeling and having it written down kept the labels consistent. Without it, those borderline posts would have been labeled inconsistently depending on mood.
