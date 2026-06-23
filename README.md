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