# who-stockout-rag-eval

A small retrieval-augmented question-answering system over five public documents on medicine shortages and supply chains, built to be **evaluated**, not just demonstrated. The point of the repo is the evaluation: a fixed question set, three separate scores per question, a failure analysis, and one documented change with a before-and-after comparison.

Author: Jennifer Ogechi Ezoh (AI governance and risk analyst). Built and run 2 to 5 Nov 2026.

## What this is, and is not

- A learning project and an evaluation exercise. It is **not** a tool for operational use in a health supply chain.
- Self-scored by the person who wrote the questions. There is no second scorer.
- 20 questions in total. With 2 to 4 questions per type, results are reported as **counts, not percentages**.

## Source documents (links only, no copies)

The PDFs are not in this repo. Please download them from the original publishers:

1. WHO report of the December 2015 technical consultation on preventing and managing global stockouts of medicines (`medicines_shortages.pdf`): [LINK]
2. WHO report of the October 2016 expert consultation on shortage and stockout definitions (`meeting_report_october_shortages.pdf`): [LINK]
3. WHO 2017 report on a global notification system for medicine shortages (`WHO-EMP-IAU-2017.15-eng.pdf`): [LINK]
4. JSI Supply Chain Manager's Handbook, chapter 5, quantification: [LINK]
5. Nigeria National Health Supply Chain Strategy and Implementation Plan: [LINK]

Licences differ and are the publishers' own. The 2016 report is marked "All rights reserved", so no text from it is republished here beyond short quotations in the results.

## System

| Component | Setting |
|---|---|
| Text extraction | pypdf, per page |
| Chunking | 1,500 characters, stride 1,300 (200 overlap), chunks do not cross pages; 354 chunks |
| Embedding model | `gemini-embedding-001` (3,072 dimensions) |
| Retrieval | cosine similarity, top k |
| Generation model | requested `gemini-3.5-flash`; the API reported `gemini-3.6-flash` as the serving model (`model_version`, 4 Nov 2026) |
| Prompt | Answer only from the excerpts, cite file and page, otherwise say exactly "The documents do not answer this." |

Page numbers are PDF page numbers (1-based), not the printed page numbers.

## Evaluation design

Each question is scored on three things separately:

1. **Answer correct** (Y / Partial / N), against a written correct answer.
2. **Citation correct**: the cited page contains the supporting text.
3. **Refusal behaviour right**: answers when it can, refuses when it cannot.

Each miss gets one failure category: extraction, retrieval miss, generation, wrong refusal, missed refusal, citation error.

Question types: direct, table/layout, aggregation, cross-document, nuance/trap, unanswerable. Questions were written before the run and are not edited after seeing results.

## Results: baseline (k=4)

11 right (including 3 correct refusals), 3 partial, 6 wrong.

| Type | Right | Partial | Wrong |
|---|---|---|---|
| Direct (4) | 2 | 0 | 2 |
| Table / layout (3) | 2 | 1 | 0 |
| Aggregation (2) | 1 | 0 | 1 |
| Cross-document (4) | 2 | 1 | 1 |
| Nuance / trap (4) | 1 | 1 | 2 |
| Unanswerable (3) | 3 | 0 | 0 |

Failure analysis: [TO WRITE AFTER k=8 IS SCORED. Key points so far: 5 of 6 wrong answers concern the October 2016 report; 4 wrong refusals; 2 confident wrong answers (Q17, Q39); 0 missed refusals.]

Per-question scores: `results_k4_scored.csv`.

## Experiment: k=8

Predictions were committed before the run (`predictions_k8.txt`). k=8 was chosen **after** the Q29 miss (the correct page ranked 6th of 354), so any improvement is exploratory, not a held-out result.

Results: [TO FILL IN: fixed / broken / unchanged counts, predictions vs outcomes.]

## Limitations

- Small sample, one scorer, questions written by the person who built the system.
- Q18 can be answered from one document, so it does not test cross-document retrieval.
- Chunks do not cross page boundaries, so answers that span pages can be split (seen in Q35).
- Multi-column tables and the annex tables of the 2016 report extract with columns spliced together.
- The free tier allows 20 generation requests per day per model, so each full run takes a day.
- The serving model name differed from the requested name.

## Reproducing

Run the notebook in Google Colab with your own Gemini API key stored in Colab Secrets. The index (`chunks.json`, `vectors.npy`) is rebuilt from the PDFs and is not included, because it contains document text.
