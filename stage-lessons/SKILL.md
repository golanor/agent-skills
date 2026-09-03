---
name: stage-lessons
description: Maintain per-stage reports and lessons files during long, multi-stage project work (research, simulation, migrations, multi-phase builds). Use when starting a staged project, completing a stage/phase, revisiting earlier conclusions, or before modifying a module that a prior stage established. Also use when the user asks to "write up this stage", "record lessons", or "revise the report".
version: 1.2.0
tags: [skill, project-memory, research, documentation, process]
---

# Stage reports & lessons

Durable project memory written *during* the run, in the repo, so any later
session (or reviewer) inherits calibration decisions, convention traps, and
retracted conclusions instead of re-deriving or re-breaking them.

## Layout

```
reports/
  INDEX.md                 # one line per stage: modules touched, headline conclusion
  stage0_<topic>.md        # findings report: what was built, key numbers
  stage0_lessons.md        # lessons: what surprised us and why it matters
  stage1_<topic>.md
  stage1_lessons.md
  ...
```

One report + one lessons file per stage/phase. An agent-instructions pointer
(e.g. `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or your tool's equivalent) is
not optional — the skill only loads when invoked, so the pointer is what makes
"read before write" happen in sessions that never invoke it. Create it in the
same change that creates `reports/`:

> Stage STOPs produce report + lessons. Before touching a module, check
> `reports/INDEX.md` for the stage that established it and read that stage's
> `*_lessons.md` — it records calibration decisions, convention traps, and
> retracted conclusions.

## Index (INDEX.md)

One line per stage, updated at every stage boundary and every revision:

```
| Stage | Modules/files established | Headline conclusion | Revised? |
|---|---|---|---|
| 0 | core/model.py, tests/test_model.py | model reproduces reference within 1% at the baseline point | — |
| 1 | pipeline/transform.py | transform is order-invariant on the test set | rev 2025-08-12 |
```

This is what makes "read before write" tractable at stage 6+ — the module
column answers "which lessons file do I need?" without guessing from topics.

## Reports (stageN_<topic>.md)

Written when a stage completes. Contents:

- What was implemented/decided, with the modules and data files it produced.
- The key quantitative results as tables — pin the exact numbers, with the
  parameter point they were computed at (seeds, reference settings).
- Explicit caveats: assumptions, known unknowns, validity limits.
- Structural findings — conclusions that change how later stages should
  work, marked as such.

Keep it to ~1–2 pages: a report pins conclusions and numbers; derivations,
plots, and exploration live in the code, notebooks, and notes it links to.
If a section is narrating *how* the work went rather than *what holds*, cut
it or move the transferable part to the lessons file.

## Lessons (stageN_lessons.md)

Not a changelog. Each entry is a numbered item: **bolded claim**, then 2–5
lines of evidence, then the transferable lesson. Record only things a
competent future session would otherwise get wrong:

- **Convention traps**: unit/scale-factor placements, sign conventions,
  off-by-one indexing — and the numerical check that settled them.
- **Calibration decisions**: what was calibrated against what, and why.
- **Retracted conclusions**: what we believed earlier, why it was wrong,
  what replaced it. Never silently delete the old belief.
- **Surprises**: results that contradicted the plan or intuition, with the
  cheap check that caught them ("the reference run cost 10 s and caught it").
- **Process lessons**: e.g. "the tests all encoded the same wrong factor —
  spec conventions propagate unquestioned into code, tests AND reports."

Example entry (the shape to imitate):

> **3. The scale factor in the reference is off by 2π from our code.**
> Stage-1 numbers were ~6.28× off against the reference table; a single-point
> comparison run (10 s) located the factor in the amplitude, not the
> detuning. Lesson: when matching an external reference, verify ONE number
> end-to-end numerically before building on the mapping — unit conventions
> don't announce themselves.

**Capture as you go.** Don't wait for the stage boundary: when a surprise,
trap, or retraction happens mid-stage, append it to the stage's lessons file
immediately (mark it `[draft]` if unvetted). Sessions compact and die; the
boundary pass then curates — merge duplicates, drop non-lessons, remove the
draft marks. A stage boundary with an empty lessons file is a smell, not a
clean run.

## Rules

1. **Read before write.** Before modifying a module, find its stage in
   `INDEX.md` and read that stage's lessons file. Treat pinned conclusions
   there as established facts — don't re-derive them; if new evidence
   contradicts one, that's a revision (rule 3), not an edit.
2. **Write at stage boundaries.** A stage isn't done until its report,
   lessons, and INDEX line exist — and, when the user is learning from the
   work (agent-written code, research stages), until the comprehension gate
   below passes. For plan-driven projects, make this the stage's STOP
   condition.
3. **Revise by appending, never rewriting.** When later work invalidates a
   report's numbers or conclusions, append a dated section —
   `## Revision YYYY-MM-DD — <what changed>` — stating what changed, why,
   old → new values, and which conclusions survive. Lessons files are
   historical records; add a matching dated revision entry there for the
   meta-lesson. The original text stays. Mark the stage as revised in
   INDEX.md.
4. **Link supersession both ways.** The revision in the old report names the
   stage that overturned it; the new stage's report has a short
   "Supersedes" line citing what it invalidated. A reader entering from
   either end must be able to find the other.
5. **Pin numbers, not adjectives.** "accuracy 0.52 → 0.626 with the extra
   feature" ages well; "significantly better" doesn't. Always include the
   parameter point.
6. **Record spec deviations.** If implementation deviates from the plan/spec
   (or amends it), flag it in both the spec (dated amendment note) and the
   stage report.
7. **Keep tests honest against reports.** When a report quotes a number a
   test also pins, changing one means changing both — mention the test in
   the report so drift is discoverable.

## Comprehension gate (with socratic-chavrusa)

When the agent did most of the building, the stage artifacts are also the
user's comprehension test — load the `socratic-chavrusa` skill and apply its
anti-deskilling rules at the boundary. The invariant: **the agent may
generate code and number tables freely, but every durable conclusion is
user-generated and agent-critiqued.**

A stage STOP additionally requires:

1. **User-drafted findings.** The USER writes the report's "structural
   findings" and "caveats" sections (the agent supplies the raw number
   tables). The agent critiques the draft — rejects vague or unsupported
   claims, demands the parameter point — but never ghost-writes these
   sections. Writing the conclusions is the test of understanding them.
2. **Teach-back on each structural finding** (chavrusa L4): why does it
   hold, and what would break it? A finding that can't survive
   interrogation isn't a finding yet — it goes back as a known unknown.
3. **For simulation/model stages — a prediction round** per socratic-chavrusa's
   Agent-Written Simulations protocol: the model card is a `reports/`
   artifact alongside the stage reports and must be current; the validation
   matrix is user-designed and passing; the user clears a prediction round
   on the stage's behavior (intervention → predict → run). Prediction
   misses are lessons entries, not just debugging.
4. **Cards from traps** (optional but default-on): convention traps and
   calibration decisions in the lessons file are ideal spaced-repetition
   material — offer the card forge on them at the boundary.

Cross-stage retrieval: at the START of stage N, run a short cold retrieval
round (chavrusa L5) on the headline conclusions of stages 0..N−1 from
INDEX.md before building on them — same pattern as chavrusa's session
priming, with lessons files as the deck. If the environment lacks
socratic-chavrusa, keep requirements 1 and 3's artifacts (user-drafted
findings, model card, validation matrix) and skip the interactive rounds.

## Adapting to a new project

- Rename "stage" to whatever the project's phases are (sprint, milestone,
  migration batch).
- If there's no `reports/` convention yet, create it — INDEX.md and the
  agent-instructions pointer in the same change (this is what enforces rule 1
  in future sessions).
- For small projects a single `LESSONS.md` with dated sections is fine; the
  append-only revision rule and capture-as-you-go still apply, and INDEX.md
  can be skipped until there's more than one file to index.
