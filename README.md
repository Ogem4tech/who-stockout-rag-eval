# who-stockout-rag-eval

A small retrieval-augmented question-answering system over five public documents on medicine shortages and supply chains, built to be **evaluated**, not just demonstrated. The point of the repo is the evaluation: a fixed question set, three separate scores per question, a failure analysis, and one documented change with a before-and-after comparison.

Author: Jennifer Ogechi Ezoh (AI governance and risk analyst). Built and run 2 to 7 Nov 2026.

## What this is, and is not

- A learning project and an evaluation exercise. It is **not** a tool for operational use in a health supply chain.
- The code was written with AI coding assistance. The question bank, the correct answers, the scoring and the failure analysis are my own work.
- Self-scored by the person who wrote the questions. There is no second scorer.
- 20 questions in total. With 2 to 4 questions per type, results are reported as **counts, not percentages**.

## Source documents (not included; download from the publishers)

1. WHO technical consultation on preventing and managing global stock outs of medicines, Geneva, 8 to 9 December 2015 (report; file used: `medicines_shortages.pdf`)
2. WHO, Technical Definitions of Shortages and Stockouts of Medicines and Vaccines, meeting report, 5 October 2016 (WHO/EMP/IAU/2017.03; file used: `meeting_report_october_shortages.pdf`)
3. WHO technical consultation meeting report, July 2017, on a global notification system for medicine shortages (WHO/EMP/IAU/2017.15; file used: `WHO-EMP-IAU-2017.15-eng.pdf`)
4. JSI, Supply Chain Manager's Handbook, chapter 5 (quantification)
5. Nigeria, National Health Supply Chain Strategy and Implementation Plan

Licences differ and belong to the publishers. The 2016 report is marked "All rights reserved", so no text from it is republished here beyond short quotations.

## System

| Component | Setting |
|---|---|
| Text extraction | pypdf, per page |
| Chunking | 1,500 characters, stride 1,300 (200 overlap), chunks do not cross pages; 354 chunks |
| Embedding model | `gemini-embedding-001` (3,072 dimensions) |
| Retrieval | cosine similarity, top k |
| Generation model | requested `gemini-3.5-flash`; the API reported `gemini-3.6-flash` as the serving model (`model_version` field, recorded for every k=8 answer) |
| Prompt | Answer only from the excerpts, cite file and page, otherwise say exactly "The documents do not answer this." |

Page numbers are PDF page numbers (1-based), not printed page numbers. Runs took place 4 to 6 Nov 2026.

## Evaluation design

Each question is scored on three things separately:

1. **Answer correct** (Right / Partial / Wrong), against a written correct answer.
2. **Citation correct**: the cited page contains the supporting text.
3. **Refusal behaviour right**: answers when it can, refuses when it cannot.

Each miss gets one failure category: extraction, retrieval miss, generation, wrong refusal, missed refusal, citation error.

Question types: direct, table/layout, aggregation, cross-document, nuance/trap, unanswerable. 41 questions were drafted, 20 were run. The questions were fixed before the run and not edited after seeing results.

## Results

| Type | k=4 right / partial / wrong | k=8 right / partial / wrong |
|---|---|---|
| Direct (4) | 2 / 0 / 2 | 3 / 1 / 0 |
| Table / layout (3) | 2 / 1 / 0 | 2 / 1 / 0 |
| Aggregation (2) | 1 / 0 / 1 | 1 / 0 / 1 |
| Cross-document (4) | 2 / 1 / 1 | 3 / 1 / 0 |
| Nuance / trap (4) | 1 / 1 / 2 | 1 / 2 / 1 |
| Unanswerable (3) | 3 / 0 / 0 | 3 / 0 / 0 |
| **Total (20)** | **11 / 3 / 6** | **13 / 5 / 2** |

Per-question scores: `results_k4_scored.csv` and `comparison_k4_vs_k8.csv`.

## Experiment: retrieving 8 chunks instead of 4

Predictions were committed before the run (`predictions_k8.txt`). k=8 was chosen **after** the Q29 miss (the correct page ranked 6th of 354), so any improvement is exploratory, not a held-out result.

**Change from k=4 to k=8 (20 questions):** 2 fixed (Q29, Q37), 2 improved from wrong to partial (Q30, Q39), 0 broken, 16 unchanged. Of the 16, Q17 changed form (a confident wrong answer became a wrong refusal) and Q35 stayed partial with a weaker 2015 half.

**Predictions against outcomes:**

| Prediction | Outcome |
|---|---|
| Q29 fixed | Correct. Page 11 was in the top 8 (rank 6). |
| Q30 and Q39 might be fixed | Partly. Both moved from wrong to partial. |
| Q17 probably not fixed | Correct. It stayed wrong. The 2015 page 14 was retrieved at rank 8 and the system still refused. |
| Q37 probably not fixed, as a prompt problem | **Wrong.** It was fixed. At k=4 the 2016 pages 19 and 20 were missing; at k=8 they arrived at ranks 5 and 8 and both halves were answered. My explanation (a prompt problem) was not supported. |
| Q38 uncertain | Stayed wrong. Pages 10, 11 and 12 were retrieved and the system still refused. |
| Risk: a passing question breaks, or an unanswerable one gets answered | Did not happen. No passing question broke and all three unanswerable questions were still refused. |

## Failure analysis

- **Retrieval depth mattered most.** In the 2016 definitions report, the pages with the answer ranked just below the top 4 (Q29 at rank 6). The margins were thin: for Q30, page 11 only made it in at rank 8, so k=7 would have missed it.
- **Wrong refusals fell from 4 to 2** (Q17 and Q38 remain). In both, the relevant page was in the retrieved set (Q17: page 14 at rank 8; Q38: pages 10 to 12) and the system still said the documents do not answer. This points to the all-or-nothing refusal instruction, which I have not yet tested separately.
- **Confident wrong answers** (Q17 and Q39 at k=4) were the most concerning failures. Neither appears as a clear wrong answer at k=8, but Q39 is still imperfect: it presents the one-day wording from the annex without saying it comes from an MSF entry.
- **Page boundaries and tables:** answers that split across pages (the result sentence in Q35 at the 2017 pages 10 and 11) and multi-column tables (Q33 omitted one field) were not helped by k=8.

## Limitations

- Small sample, one scorer, questions written by the person who built the system, and k=8 chosen after one observed failure.
- **Each configuration was run once.** The model's answers vary between runs, so some of the differences (for example the weaker 2015 half in Q35) may be run-to-run variation and not an effect of k. I did not repeat runs.
- Q18 can be answered from one document, so it does not test cross-document retrieval.
- Chunks do not cross page boundaries. Multi-column tables and the 2016 annex extract with columns spliced together.
- The free tier allows 20 generation requests per day per model, and failed requests appear to count toward it, so a full 20-question run took about two days. The quota resets at about 09:00 Lagos time.
- The serving model name differed from the requested name.
- Details added by k=8 answers beyond the correct answers (for example Q15 and Q36) were not checked against the PDFs.

## Next experiment (not run)

Change the refusal instruction so the system answers the part the excerpts support and says what is missing. This should be tested as its own change, with all 20 questions re-run on both versions.

## Reproducing

Run the notebook in Google Colab with your own Gemini API key stored in Colab Secrets. The index (`chunks.json`, `vectors.npy`) is rebuilt from the PDFs and is not included, because it contains document text.
