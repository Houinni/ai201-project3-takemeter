# AI Usage Log — TakeMeter

Running record of AI tool use on this project. Kept *as work happens* so the README "AI Usage" section (graded: ≥2 specific instances — what I directed, what I revised/overrode + any annotation assistance) can be written accurately at the end.

**Format per entry:** date · tool/model · what I directed it to do · what it produced · what I kept / changed / overrode.

---

## Entry 1 — Label design (Milestones 1–2)
- **Date / tool:** 2026-06-26 · Claude (Opus 4.8)
- **Directed it to:** react to real r/nba comments without a preset scheme and help surface natural label categories; then pressure-test a draft 4-class scheme by rating individual comments.
- **Produced:** per-comment ratings with reasoning, and a candidate scheme (`not-a-take` / `noise` / `decent` / `insightful`) plus boundary-resolving rules.
- **What I kept / changed / overrode:** kept the 4-class structure and the strip-and-judge + specificity rules. **Overrode / owned myself:** the final label definitions and the unit-of-analysis decision (comment + parent context). Set the accuracy policy (grade reasoning, not factual correctness) as my own call, not the model's.

## Entry 2 — Rubric stress-test (Milestone 2, §8.1)
- **Date / tool:** 2026-06-26 · Claude (Opus 4.8)
- **Directed it to:** generate ~8 synthetic r/nba comments deliberately sitting on each label boundary, then classify each using only my rubric.
- **Produced:** 8 boundary comments (see `rubric_stress_test.md`); 6 resolved cleanly, 2 wobbled.
- **What I kept / changed / overrode:** used the 2 wobbles to **add** the mockery rule (sarcasm w/o a claim = `not-a-take`) and **rewrite** the decent↔insightful test into an operationalized 3-part trigger. **Guardrail I imposed:** the synthetic comments are excluded from the training/test data to avoid distribution contamination — the model's output was used for *definition-testing only*.

---

## Pending entries (fill in as I do them)

## Entry 3 — Annotation assistance (Milestone 3, §8.2)
- *To complete if/when I use an LLM to pre-label.* Record: tool/model, how many items pre-labeled, agreement rate with my human labels, and how many I corrected. Every pre-labeled row is flagged `prelabeled_by_llm=true` in the dataset; the human label is always ground truth.

## Entry 4 — Failure-pattern analysis (Milestone 6, §8.3)
- *To complete after evaluation.* Record: what error list I gave the model, which patterns it proposed, and which I confirmed (with quantification) vs. discarded against the actual misclassified examples.
