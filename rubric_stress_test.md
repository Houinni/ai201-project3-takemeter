# Rubric Stress-Test (Milestone 2, §8.1)

**Purpose:** before annotating 200 real examples, generate synthetic r/nba comments that deliberately sit *on* each label boundary, then try to classify each using only the rubric in `planning.md`. Any comment that can't be classified cleanly means a definition is too loose — tighten it now.

**Tool:** Claude (Opus 4.8), 2026-06-26.
**Guardrail:** these comments are synthetic and exist for definition-testing ONLY. They are **never** added to the training/test CSV — synthetic text doesn't match the real distribution and would contaminate evaluation.

## Generated comments + rubric verdicts

| # | Boundary | Comment | Verdict | Clean? |
|---|---|---|---|---|
| 1 | not-a-take ↔ noise | "This. Every year it's the same story with this front office." | strip "This" → unsupported complaint → **noise** | ✅ |
| 2 | not-a-take ↔ noise | "lmaooo not the 'he's just 22' excuse again" | mocks a position, no counter-claim → **not-a-take** | ❌ wobbled |
| 3 | noise ↔ decent | "Duren was bad cause he's got no motor in the playoffs, simple as" | "no motor" = vague trait, not checkable → **noise** | ✅ |
| 4 | noise ↔ decent | "They should've staggered Cade's and Duren's minutes so one was always out there to run P&R" | specific tactical reason → **decent** | ✅ |
| 5 | decent ↔ insightful | "Duren's rebounding fell in the playoffs because teams started boxing him out instead of just outjumping him like in the regular season" | specific mechanism + opponent adjustment → **insightful** | ⚠️ borderline |
| 6 | decent ↔ insightful | "The real issue isn't Duren, it's Detroit has zero spacing so he's catching everything in a crowd" | "no spacing" = cause a casual fan names → **decent** | ⚠️ borderline |
| 7 | insightful vs wrong | "They literally can't match a max sheet because Duren's base-year compensation, so his outgoing salary only counts half" | grade structure not truth; if I can't verify BYC applies → **unsure** | ❌ competence-dependent |
| 8 | insightful vs wrong | "Offer sheets don't matter, Detroit has full Bird rights so they go over the cap regardless" | has reasoning (Bird → over cap); grade on structure → **decent** | ⚠️ |

## What wobbled → rubric changes made

**Wobble A — sarcasm/mockery (#2).** The rubric didn't say whether dismissive snark is `not-a-take` or `noise`.
→ Added the **mockery rule** to `planning.md` §3: pure sarcasm that ridicules a take without advancing its own checkable claim = `not-a-take`; only `noise` if a stand-alone assertion remains after stripping the snark.

**Wobble B — decent↔insightful (#5, #6).** "Could I have generated it?" was too subjective to apply consistently.
→ Replaced with the **operationalized test** in §3: `insightful` requires (a) a specific causal mechanism a casual viewer wouldn't state, (b) non-obvious domain knowledge, or (c) a genuinely novel reframe; causes a regular fan reaches for (spacing, youth, effort, "no motor") stay `decent`. Under this, #5 → insightful, #6 → decent.

**#7/#8 — accuracy boundary.** Confirmed the existing accuracy policy works (grade structure, not correctness) but both hinge on annotator competence: if I can't tell whether a cap-mechanics claim coheres, tag `unsure` rather than bluff `insightful`. No rule change; flagged as a real risk for my own annotation.

## Outcome
6/8 resolved cleanly on first pass; the 2 wobbles produced 2 concrete rule additions. Rubric considered tightened enough to begin Milestone 3 annotation.
