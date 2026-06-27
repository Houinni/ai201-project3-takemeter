# TakeMeter — Planning

## 0. One-paragraph description

r/nba is one of the largest sports communities on Reddit, where fans react to games and trades in real time and the discourse swings constantly between sharp basketball analysis and pure tribal hyperbole. TakeMeter classifies each comment as `not-a-take`, `noise`, `decent`, or `insightful` — separating jokes and bare assertions from opinions that are actually backed by specific basketball reasoning. These distinctions matter because the community's value lives in the rare comment that explains *why* something happened (a coverage scheme, a cap mechanic), and that signal is constantly buried under volume; surfacing it is what separates a useful discussion thread from noise.

---

## 1. Community

**Choice: r/nba** (data drawn from daily discussion threads, post-game threads, and comments on discussion posts).

**Why this community.** I can judge basketball takes confidently without being a specialist, which matters because I am the primary annotator and *the labels can never be better than the annotator's judgment*. NBA takes are also legible to a general audience in a way that, say, music-theory or competitive-game takes are not, so a second annotator can be onboarded with a short rubric.

**Why it's a good fit for classification — the discourse is genuinely varied.** A single post-game thread contains the full quality spectrum within minutes of each other:
- **Pure noise / hyperbole:** "LeBron is the most overrated player ever," "Refs rigged it 😤."
- **Jokes, questions, co-signs** that aren't opinions at all.
- **Solid conventional analysis:** "They lost because they kept switching their bigs onto Curry."
- **Genuinely insightful, mechanism-level takes:** cap-mechanic arguments, coverage-scheme breakdowns.

This range is exactly what makes the task non-trivial: the classes are real and distinguishable, but the boundaries between adjacent classes are fuzzy enough to force hard annotation decisions. High volume (thousands of text comments/day) also makes data collection trivial, and the value proposition is concrete — the good stuff is constantly buried under the noise.

**Unit of analysis.** One item = a single comment, judged *with its parent (comment or post) attached as context*. Replies on r/nba are routinely uninterpretable in isolation (e.g. a comment starting "Exactly…"), so context is mandatory for both the annotator and the model.

---

## 2. Labels

Single-label scheme: each item gets exactly **one** of the four tags below. (`unsure` exists but is an annotator-only escape hatch — see §3 — never a model class.)

### `not-a-take`
**Definition:** A comment that asserts no evaluative basketball opinion — a joke, question, score update, pure agreement, or off-topic/meta chatter.
- *"What time's the game on tonight?"*
- *"Bro your flair is sending me 💀"*

### `noise`
**Definition:** A basketball opinion with no supporting reasoning — pure hyperbole, tribal assertion, or a bare claim.
- *"LeBron is the most overrated player in history."*
- *"Duren is straight trash, just cut him."*

### `decent`
**Definition:** A real opinion backed by at least one specific basketball reason, but expressing the conventional or obvious take.
- *"dk man he was 22 going up against 25yo Mobley and 28yo Jarrett Allen in his first playoffs with piss poor spacing to work with."*
- *"They lost that series because they kept switching their bigs onto Curry and never adjusted."*

### `insightful`
**Definition:** A specific, non-obvious, well-justified take that tells you something you couldn't have generated yourself.
- *"Honestly could see this as a compromise in the next CBA — make signed offer sheets binding but not added to the cap until after the FA moratorium, so small markets keep their drafted players and rival teams can still use low cap holds."*
- *"Duren's playoff drop wasn't effort — when the Cavs went to drop coverage his rolls stopped generating rim pressure, so his rebounding fell because he was starting boxouts 4 feet further from the rim."*

---

## 3. Hard edge cases

Every adjacent boundary produces genuinely ambiguous comments. Each case below names the ambiguity and the **mechanical rule** I'll apply when I hit it during annotation. The point of writing rules now is that a rule applied consistently beats a gut call made differently on Tuesday than on Friday.

**`not-a-take` ↔ `noise` — the co-sign + anecdote.**
*Example:* "Exactly. Half the fanbase is already talking about trading Duren and three firsts for Jaylen Brown." It opens as pure agreement but tacks on a hyperbolic observation that *implies* an opinion.
**Rule (strip-and-judge):** Remove leading agreement markers ("Exactly," "This," "Facts") and emoji, then judge what remains. Nothing propositional left → `not-a-take`; an unsupported claim remains → `noise`.

**`noise` ↔ `decent` — the sliver of a reason.**
*Example:* "Duren is washed, dude vanished against any real center." "Vanished against real centers" looks like a reason but is just a vaguer restatement of the insult.
**Rule (specificity test):** A reason only counts if it's specific and checkable (a named matchup, stat, or mechanism). Vague restatements don't promote → stays `noise`.

**`decent` ↔ `insightful` — the polished obvious take.** *(Expected to be the worst boundary for agreement.)*
*Example:* the long Duren-$287M post — specific, balanced, well-written, but the core framing ("pay for the regular season vs. fear the playoff version") is the obvious one.
**Rule (could-I-have-generated-it test):** `insightful` = a point I could not have produced myself; `decent` = a good version of the point any informed fan reaches. Length and good writing do **not** promote (guard against the "longer = better" confound).

**`insightful` vs. confidently-wrong — the accuracy question.**
*Example:* "Detroit can't offer that because his cap hold would balloon their Bird rights past the second apron." Sounds expert; may be false.
**Rule (accuracy policy):** Grade *reasoning quality, not factual correctness* — a well-structured argument on a false premise is still judged on structure. **But** if I lack the domain knowledge to tell whether the reasoning even coheres, I tag `unsure` rather than guess upward.

**The `unsure` escape hatch.** When genuinely torn, tag `unsure` → set aside → adjudicate in batches against the rules above. These items never enter training as a class; forcing a call on truly ambiguous items just injects noise into the labels.

**Cross-cutting rules (every item):** (1) casual/profane tone doesn't demote a comment with real reasoning; (2) strip agreement markers/emoji before judging; (3) judge with parent attached; (4) no length or formatting bonus.

---

## 4. Data collection plan

**Source.** Reddit API via PRAW, from r/nba: daily discussion threads, post-game threads, and top-level comments on discussion/analysis posts. For each item I store: comment text, **parent text**, post title, score, comment id, permalink, timestamp. I'll pull from many threads (~20–50) across multiple game days so the sample isn't dominated by one game or team.

**Scrape vs. label are separate steps.** Collect a large raw pool (~3,000–5,000 comments) automatically; then **randomly sample** items to hand-label. Random sampling keeps the label distribution honest and leaves a large unlabeled remainder for the test set and error analysis.

**Targets.** Aim for **~300 labeled items** total, with a floor of **~50 per class** so even the rare classes are learnable. A purely random sample will be badly imbalanced (most comments are `not-a-take`/`noise`), so the plan is a hybrid: label one **random** batch first to measure the natural distribution, then **targeted top-up** for the starved classes.

**If a label is underrepresented after 200 examples** (most likely `insightful`):
1. **Targeted sampling, not synthetic data** — pull more candidates from where that class concentrates (long comments, analysis-post threads, high-effort flaired posts) and label those. This biases the *sampling* but keeps every label a real human judgment.
2. **Class weighting / oversampling at train time** rather than forcing a balanced corpus, so the model still sees the true prior.
3. **If still too thin to learn (e.g. < ~30 clean examples), collapse the scheme** — merge `decent`+`insightful` into a single "substantive" class for a 3-class task — and report this as a finding. A reliable 3-class model beats an unreliable 4-class one.
4. Report the final per-class counts and the natural distribution explicitly; imbalance is a result, not something to hide.

**Inter-annotator check.** Double-label ~100 items with a second annotator (or myself two weeks apart), compute agreement, and rewrite the rubric where disagreements cluster *before* scaling up.

---

## 5. Evaluation metrics

Accuracy alone is disqualified up front: with a heavily imbalanced corpus, a model that predicts `not-a-take` for everything could score high accuracy while being useless. The metrics below are chosen around the imbalance and around *which errors actually hurt*.

- **Macro-F1 (primary).** Averages F1 across classes equally, so the rare-but-important `insightful` class counts as much as the dominant `not-a-take`. This is the headline number.
- **Per-class precision / recall + confusion matrix.** I need to see *where* it fails, not just an aggregate. The confusion matrix also reveals whether errors are "near" (decent↔insightful) or "far" (insightful↔not-a-take).
- **Quadratic Weighted Kappa (QWK).** The classes are roughly ordinal (`noise` < `decent` < `insightful`), so not all mistakes are equal — calling an `insightful` comment `not-a-take` is far worse than calling it `decent`. QWK penalizes distant errors more and credits near-misses, which plain F1 ignores.
- **Model agreement vs. human ceiling.** I'll compare model-vs-human agreement to the **human-vs-human** agreement measured in §4. Human agreement is the ceiling — a model that matches it is as good as the task allows, and a model far below it has headroom. Reporting the model number without the human ceiling is meaningless.
- **`insightful` precision and recall, called out separately.** This is the deployment-critical class (see §6), so it gets its own line rather than being averaged away.
- **Precision@k for the surfacing use case.** If the real product is "show the top 10 takes in this thread," what matters is whether the items it *ranks highest* are actually good — measured as precision@10 on held-out threads.

**Baselines to beat** (so the numbers mean something): a majority-class baseline, and a simple length + keyword logistic-regression baseline. The fine-tuned model has to clearly beat both.

---

## 6. Definition of success

The product goal is **surfacing**: pull the genuinely good takes out from under the noise. That shapes what "good enough" means — for a surfacing tool, **precision on the good classes matters more than catching every last one.** Concrete, checkable targets:

**Genuinely useful (target):**
- **Macro-F1 ≥ 0.55** on the 4-class task, and **≥ 10 absolute F1 points** above both baselines.
- **Model-vs-human QWK ≥ 0.60 absolute, and ≥ 0.90 × the human-vs-human QWK** (i.e. at least 90% of the way to the human ceiling).
- **`insightful` precision ≥ 0.70** — when it says a comment is insightful, it's right ≥ 70% of the time (false promotions are what destroy trust in a surfacing tool).
- **`not-a-take` recall ≥ 0.85** — reliably filters junk, the easiest and highest-volume win.

**Good enough to deploy in a real community tool (e.g. a "top takes of this thread" widget):**
- **Precision@10 ≥ 0.80** on held-out threads — of the 10 comments it surfaces as best, ≥ 8 are genuinely `decent`/`insightful` to a human. A reader skimming the widget should rarely hit a dud.
- `insightful` **recall ≥ 0.50** — it's acceptable to miss some gems (recall), but not to surface garbage (precision); in a high-volume feed, missing half the gems still surfaces plenty.
- No catastrophic confusions: the rate of `insightful` → `not-a-take`/`noise` (and vice-versa) stays low in the confusion matrix.

**Acceptable fallback:** if the 4-class `decent`/`insightful` split proves unlearnable (§4), a **3-class** model (`not-a-take` / `noise` / `substantive`) meeting the analogous macro-F1 and precision@10 bars is a legitimate success and an honest finding about the limits of the label design.

---

## 7. Self-review — are the success criteria objectively checkable?

Each §6 target maps to a single computable number on a held-out test set, so at the end I can state pass/fail unambiguously:

| Criterion | How it's decided | Objective? |
|---|---|---|
| Macro-F1 ≥ 0.55 & +10 over baselines | compute macro-F1 for model and both baselines | ✅ yes |
| QWK ≥ 0.60 and ≥ 0.90× human ceiling | compute QWK; human ceiling measured in §4 | ✅ yes (requires the human double-label set to exist) |
| `insightful` precision ≥ 0.70 | per-class precision from confusion matrix | ✅ yes |
| `not-a-take` recall ≥ 0.85 | per-class recall | ✅ yes |
| Precision@10 ≥ 0.80 | rank held-out threads, human-check top 10 | ✅ yes, but needs a small fresh human judgment pass at eval time |

**Gaps I'm closing:** (1) QWK-vs-ceiling is only meaningful if the human inter-annotator set is actually collected — so that double-labeling in §4 is a hard prerequisite, not optional. (2) Precision@10 requires a human to grade the model's top-10 at eval time; I'll reserve a handful of unlabeled threads specifically for this so the test isn't run on training data. (3) All thresholds are reported **with the test-set size and per-class counts**, since an F1 from 12 `insightful` examples is not the same claim as one from 60 — small-sample numbers will be flagged as provisional.

Conclusion: the criteria are specific and falsifiable; the only dependencies are that the human-agreement set and a held-out surfacing set get built, both of which are already in the §4 plan.

---

## 8. AI tool plan

There's no application code to generate in this project, so AI tools earn their place at three specific points in the *label-design and analysis* workflow — not in implementation. Each use below is logged for disclosure in the AI usage section: I'll keep an `ai_usage_log.md` recording the tool, date, the prompt's intent, and what I did with the output.

### 8.1 Label stress-testing (before annotating)
**Use:** Paste the §2 definitions and §3 edge cases to an LLM and ask it to **generate 5–10 synthetic r/nba comments that deliberately sit on each boundary** — `not-a-take`↔`noise`, `noise`↔`decent`, and especially `decent`↔`insightful`. Then I try to classify each one *using only the rubric*.
**Decision rule:** If I can't classify a generated comment cleanly, the rubric is too loose — I tighten the definition or the resolving rule **now**, before annotating 200 real examples. I'll re-run the generation after each tightening until the boundary comments resolve consistently.
**Guardrails:** These synthetic comments are for *stress-testing definitions only* — they are **never** added to the training or test data (synthetic text doesn't reflect the real distribution and would contaminate evaluation). They live in a separate `rubric_stress_test.md`.

### 8.2 Annotation assistance (LLM pre-labeling)
**Decision: yes, but as a second opinion, not a labeler.** I'll hand-label the full set myself; for a subset I'll *also* get an LLM pre-label and use disagreements as a review trigger, not as ground truth.
- **Tool:** Claude (note exact model/version in the usage log).
- **Workflow:** label by hand first (or in parallel without seeing the model's guess), then compare. Where the human and LLM labels disagree, re-examine that item against the rubric — these are efficient flags for genuinely ambiguous cases.
- **Tracking / disclosure:** every row carries a `prelabeled_by_llm` boolean and an `llm_label` column, so it's always recoverable which items the model touched and whether my final label matched. The **ground-truth label is always the human one.**
- **Hard rule:** no item is trained on with the LLM's label as its sole source of truth. If I ever skip human review for speed, those rows are marked and excluded from the test set so evaluation stays honest.

### 8.3 Failure analysis (after evaluation)
**Use:** Export the model's wrong predictions (text + parent + true label + predicted label + confidence) and give the LLM the list, asking it to **propose patterns** in the errors before I write up the evaluation.
**What I'll look for:**
- Systematic confusions (e.g. is `decent`→`insightful` the dominant error, matching the boundary I predicted in §3?).
- Feature confounds — is it over-promoting long or jargon-heavy comments? (the "longer = better" trap from §3).
- Whether sarcasm/inside-jokes cluster in the `not-a-take`↔`noise` errors.
- Annotator-competence failures — `insightful` errors on cap-mechanics comments I may have mislabeled myself.
**Verification (mandatory):** the LLM only *proposes* patterns; I confirm each one by going back to the actual misclassified examples and checking the pattern holds — and I quantify it (e.g. "62% of `insightful` false-negatives are CBA-mechanics comments"). Any pattern I can't verify against real examples doesn't go in the writeup. The model finds hypotheses; the data confirms them.

---

## Appendix — pipeline summary

```
1. PRAW scrape ~3–5k comments (text + parent + title + score + ids) → CSV
2. Random sample → measure natural class distribution
3. Hand-label ~300 (≥50/class via targeted top-up); use `unsure` freely
4. Double-label ~100 → Krippendorff α / QWK → rewrite rubric where it cracks
5. Fine-tune classifier
6. Evaluate: macro-F1, per-class P/R, confusion matrix, QWK vs human ceiling,
   insightful P/R, precision@10 — against majority + length/keyword baselines
7. Error analysis: where it works, where it falls apart
```
