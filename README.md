# Module 7 Week B — Integration Task: Summarization & Integrated Evaluation Report

This is the starter repo for the Module 7 Week B Integration Task. **The integrated evaluation report you produce here is the M7 deliverable.**

The full integration guide is at <a href="https://levelup-applied-ai.github.io/aispire-14005-pages/modules/module-7/496c1c2b" target="_blank">the integration guide page</a> — read it first.

## Quick start

```bash
pip install -r requirements.txt
make summarize    # runs full pipeline; first run downloads ~250 MB

```

The first call to `pipeline("summarization", ...)` downloads the model. Plan ~3 minutes for the first run; subsequent runs use cached weights. The full evaluation on 120 articles completes in ~6–8 minutes on CPU after the model is cached.

## What you will produce

Committed:

* `summarize.py` — your implementation
* Updated `README.md` — 1–2 paragraphs documenting model id, corpus version, re-run command (this section is the template; replace it)
* `summary_predictions.csv` — 120 rows with reference, predicted, and per-summary ROUGE
* `summary_metrics.json` — aggregate ROUGE-1/2/L F1
* `integrated-evaluation-report.md` — six-section integrated report (the M7 deliverable). Includes an optional Section 7 (Challenge Extensions) for learners completing challenge tiers — see the integration's learner guide.

**No model file** — pre-trained model loads from Hugging Face Hub at runtime.

### Reproducibility and System Production Documentation

The primary pipeline configuration utilizes `sshleifer/distilbart-cnn-6-6` via the Hugging Face Transformers library API infrastructure. This architecture functions as an optimized, distilled transformer framework inheriting standard BART sequence-to-sequence conditional generation properties, specifically exposed to downstream abstractive summarization text patterns through comprehensive prior training on the high-density CNN/DailyMail dataset.

The evaluation suite calculates metrics against a target verification slice consisting of exactly 120 technology and entertainment news prose articles, matching them programmatically to the curated ground-truth targets stored within `data/tech_news_summaries_reference.csv`. Runtime text generation logic relies on deterministic decoding configurations combined with multi-path beam search strategies (`num_beams=4`), providing reliable sequence reproducibility across execution boundaries. The specific execution metrics compiled locally on this evaluation partition are detailed below for immediate integration benchmarking:

* **Mean Aggregate ROUGE-1 F1:** 0.3689 (validating word-level unigram presence)
* **Mean Aggregate ROUGE-2 F1:** 0.1589 (validating bigram sequence extraction stability)
* **Mean Aggregate ROUGE-L F1:** 0.2661 (validating longest common sequence structure preservation)

To clean out current output dependencies, reset infrastructure components, and completely re-trigger the automated pipeline to execute evaluation tracking over the entire dataset partition, utilize the project Makefile target abstraction layers or execute the baseline command script inside your active environment console:

```bash
python summarize.py

```

## Data

* `data/tech_news_articles.csv` — 1,033 tech / entertainment / digital-culture news articles, curated from glnmario/news-qa-summarization. The full pool is here for inspection and stretch use; the integration evaluates on the 120-article subset that has reference summaries.
* `data/tech_news_summaries_reference.csv` — 120 reference summaries (one per evaluated article), shipped with the curated dataset (CNN editor-authored summaries from the source dataset).
* `data/tiny_articles_smoke.csv` + `data/tiny_refs_smoke.csv` — 3-row CI smoke fixtures (articles and references in separate files, matching the real-data schema).

## Make targets

```bash
make summarize    # full pipeline against the 120-article evaluation set
make smoke        # CI-only target — 3-row fixture
make clean        # remove generated outputs

```

## Submission

Open a Pull Request from your working branch into `main`. The autograder runs `make smoke` against the 3-row fixture and validates artifact schemas. PR description requirements are in the integration guide.

---

## License

This repository is provided for educational use only. See [LICENSE](https://www.google.com/search?q=LICENSE) for terms.

You may clone and modify this repository for personal learning and practice, and reference code you wrote here in your professional portfolio. Redistribution outside this course is not permitted.
