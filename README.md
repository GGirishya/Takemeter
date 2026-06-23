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
