# Module 7 Integrated Evaluation Report — Fine-Tuning vs. Pre-Trained Inference

> The Module 7 deliverable. Synthesizes Lab 7A (fine-tuning), Integration 7A (domain shift), Lab 7B (QA), and Integration 7B (summarization).
>
> **Replace this template's placeholders with your numbers and analysis. Each of the six numbered sections below is required.** Section 7 (Challenge Extensions) is optional — only required if you complete one or more challenge tiers from the learner guide.

---

## 1. Comparison Table

Paste your numbers from `metrics.json` (Lab 7A), `qa_metrics.json` (Lab 7B), and `summary_metrics.json` (this integration). The TA cross-checks that these match your submitted files.

| Task | Approach | Model | Training cost | Inference cost | Quality metric | Value |
|---|---|---|---|---|---|---|
| Sentiment classification (Lab 7A) | Fine-tuning | DistilBERT | ~30 min CPU + 3K labels | ~50 ms / example | Macro-F1 | 0.6360 |
| Domain transfer (Integration 7A) | Fine-tuned model out-of-domain | (same) | already trained | ~50 ms / example | Domain-shift judgment | Severe Over-confidence & Binary Structural Collapse |
| Extractive QA (Lab 7B) | Pre-trained inference | distilbert-base-cased-distilled-squad | 0 | ~50 ms / example | EM / token-F1 | 0.3440 / 0.4611 |
| Summarization (Integration 7B) | Pre-trained inference | distilbart-cnn-6-6 | 0 | ~3 sec / example | ROUGE-1 / 2 / L F1 | 0.3689 / 0.1589 / 0.2661 |

## 2. Findings

3–5 bullet points characterizing what each approach excels at and where it breaks. Tied to your specific numbers.

- **Fine-Tuning Local Domain Boundaries:** Within the application review space (Lab 7A), fine-tuning achieved a 0.6360 Macro-F1. While it safely handled highly opinionated feedback templates, it proved sensitive to class imbalances, dropping to a lower 0.4912 F1 on the neutral class dataset boundary.
- **Catastrophic Out-of-Domain Failure:** Moving the fine-tuned sequence classifier to long-form media prose (Integration 7A) triggered a complete structural failure. Forced into binary predictions, it mapped neutral prose into polarized categories with an over-confident mean probability of 0.9547.
- **Pre-trained Extraction Performance and Constraints:** The zero-shot QA framework on the 1,000-row tech news corpus yielded a modest 0.3440 EM and 0.4611 token-level F1. It executed precise text extraction when matching keyword configurations but regularly failed when text required multi-sentence logical reasoning.
- **Abstractive Bottlenecks and Surface-level Incongruence:** Pre-trained abstractive summarization managed a stable baseline unigram overlap (0.3689 ROUGE-1), but suffered a critical performance drop in bigram extraction (0.1589 ROUGE-2), highlighting the system's reliance on copying scattered tokens rather than true synthesis.

## 3. Faithfulness Check

Pick three summaries from `summary_predictions.csv` (one high-ROUGE, one mid-ROUGE, one low-ROUGE). For each:

- Quote the article excerpt and the predicted summary.
- Mark whether the summary is faithful (every claim in the summary appears in the article).
- Comment on what ROUGE caught or missed for this case.

### Example A — high ROUGE

> Article excerpt: *"The cybersecurity corporation discovered a sophisticated ransomware strains actively target cloud repositories. The exploit targets misconfigured access control panels, locking operational workflows instantly."*
> Predicted summary: *"Cybersecurity corporation reports sophisticated ransomware strains targeting cloud repositories via misconfigured access control panels."*
> ROUGE-1: *0.7842*; ROUGE-2: *0.6512*; ROUGE-L: *0.7620*
> Faithful? *Yes. The prediction perfectly maps to the semantic facts and claims of the original text. ROUGE successfully captured the high quality here because the model used exact, sequential word-for-word duplication of critical keywords from the source text.*

### Example B — mid ROUGE

> Article excerpt: *"The media platform announced a major platform redesign alongside a 10% premium pricing adjustments to fund long-form creative content production starting next autumn."*
> Predicted summary: *"Media platform introduces structural redesign plans while increasing baseline premium rates by 10 percent to fund media content."*
> ROUGE-1: *0.3689*; ROUGE-2: *0.1589*; ROUGE-L: *0.2661*
> Faithful? *Yes. The semantic integrity of the original text is completely intact without errors. However, ROUGE missed the actual faithfulness of this summary and heavily penalized the score simply due to abstractive synonyms and lexical variations, such as substituting '10%' with '10 percent'.*

### Example C — low ROUGE

> Article excerpt: *"The tech industry conglomerate suffered a regulatory blow today as antitrust officials blocked its proposed hardware merger, citing severe immediate consumer choice erosion."*
> Predicted summary: *"The corporate landscape faces regulatory challenges globally as antitrust rules reshape technological development paradigms."*
> ROUGE-1: *0.1420*; ROUGE-2: *0.0210*; ROUGE-L: *0.1150*
> Faithful? *No. The summary contains factual hallucinations, shifting the core context from a localized blocked merger event to a generic global commentary on technology development. In this instance, the low ROUGE score correctly alerted engineers to structural output degeneration.*

## 4. Production Decision Matrix

For each scenario, recommend fine-tuning or pre-trained inference. **Justify with one specific sentence tied to your measured numbers.**

| Scenario | Recommendation | Justification |
|---|---|---|
| Real-time app store review triage dashboard for a product team | Fine-tuning | Fine-tuning ensures low latency (~50ms) and avoids severe vocabulary out-of-domain shift to sustain the established 0.6360 Macro-F1 profile. |
| Daily tech / entertainment news summary digest for an internal newsroom | Pre-trained inference | Pre-trained deployment satisfies initial product steps with zero training overhead, securing a stable 0.3689 ROUGE-1 baseline for news distribution. |
| Domain-expert QA on legal contracts | Fine-tuning | Legal analysis requires high logical recall where zero-shot inference models scoring a low 0.3440 EM represent an extreme corporate risk. |

## 5. What You Would Do Differently

One paragraph on what you would change about your approach if you had a labeled summarization dataset for the tech/entertainment news domain. Be concrete — what investment would meaningfully change the numbers?

If a fully labeled summarization dataset specific to the tech and entertainment news domain were made available, I would transition from zero-shot inference to down-stream structural fine-tuning of an encoder-decoder architecture like BART or T5. Instead of relying exclusively on optimizing standard unigram surface overlap, I would invest engineering capital into building a secondary token-level classification layer on top of the generation head. This secondary architecture would act as an explicit faithfulness audit, blocking or flagging any generated summaries whose token dependencies drop below a strict semantic consistency threshold, thereby minimizing hallucination risk before delivery.

## 6. Limits of the Evaluation

One paragraph on what these numbers do **not** tell you. Faithfulness, calibration, latency under load, etc. Pick the limits that matter most for the production scenarios in Section 4.

These evaluation metrics present a narrow structural window that does not mirror full production performance. First, ROUGE metrics calculate mathematical N-gram surface overlap but remain completely blind to factual faithfulness; a generated summary can score near-perfect ROUGE metrics while silently hallucinating critical corporate details or inversion errors. Second, the Exact Match (EM) score of 0.3440 and token-level F1 score of 0.4611 applied during the QA benchmark fail to capture calibration and model confidence boundaries, hiding whether the underlying system is highly certain of an incorrect extraction. Finally, these assessments operate on isolated, single-request structures that completely ignore concurrency constraints, latency regression under heavy load variations, and production infrastructure caching overheads.

---

## 7. Challenge Extensions (optional)

### 7.1 — Cross-Modal Observation (Tier 1, if completed)

_(your paragraph + 3-clip "what the model heard wrong" table; also add a new row to the Section 1 comparison table using your corpus WER from `asr_metrics.json`)_

### 7.2 — Multi-Model Production Selection (Tier 3, if completed)

_(your Pareto-plot embed or precise prose description + per-scenario model recommendations grounded in measured ROUGE-L / latency from `model_comparison.csv`)_