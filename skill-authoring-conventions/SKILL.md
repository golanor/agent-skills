---
name: skill-authoring-conventions
description: House rules for writing and revising agent skills (SKILL.md files). Load whenever authoring a new skill, revising an existing one, synthesizing a skill from a book or paper corpus, or reviewing a skill's design. Distilled from Matt Pocock's writing-for-agents, book-to-skill, obra/superpowers (all MIT), Ink & Switch's tools-for-thought research philosophy, and Daniel Litt's working-mathematician lessons, plus this collection's own norms.
version: 1.3.0
tags: [skill, meta, skill-design, authoring, conventions]
---

# Skill-Authoring Conventions

Rules for writing skills that steer an agent reliably without bloating its context. Apply them when creating or revising any SKILL.md.

## Structure

- **Single source of truth.** A rubric, protocol, or definition lives in exactly one skill; every other skill points at it. Duplicated meaning inflates its prominence and drifts. When two skills need the same content, one owns it and the other says "load X".
- **Router skills are legal and cheap.** A skill body can be one line delegating to another skill. Composition over duplication (Pocock's `grill-me` is a one-line router to `grilling`).
- **Progressive disclosure — two loads.** Distinguish *context load* (tokens the agent always reads) from *cognitive load* (the human's index of what exists). Inline only what every use of the skill needs; push reference material (formats, rubrics, mechanics) into sibling files reached by relative links, named `*-FORMAT.md` / `reference/*.md`.
- **Frontmatter**: `name`, `description` (trigger-front-loaded — the invocation phrases come first, terse), `version`, `tags`. Descriptions are matching surface, not prose.

## Language

- **Prompt the positive.** Steering by prohibition drags the forbidden behavior into context ("don't think of an elephant"). State what to do; keep anti-pattern lists short and secondary, each entry paired with the positive rule it violates.
- **Leading words.** Anchor each core behavior to one compact, repeatable token (*frontier*, *close the loop*, *fact/decision boundary*) and reuse the token, never a re-explanation. Repeated as a token, never as a sentence.
- **One term per concept.** Sweep synonyms — a second name for the same thing reads as a second thing.
- **Prune no-ops.** For every sentence ask: does it change behavior versus the default? Delete whole sentences that fail. Sediment accumulates from every past edit; hunt it on each revision.
- Short, imperative, second person.

## Behavior design

- **How, never whether.** A tool for thought changes *how* the user thinks and must never change *whether* they think (van Hardenberg, Ink & Switch). This is the acceptance test for every skill in this collection: name what cognitive work the user still does when the skill runs. If the answer is "none", the skill fails.
- **A verifier closes every loop.** Code the agent writes passes through typechecker, linter, and tests before the user sees it; prose and reasoning get "a stream of text back." Close that asymmetry: every skill output passes a verifier before it counts — a mechanical one where it exists (sanity checks, proof checks, citation resolution), and the user where none does (prediction before reveal, teach-back, validation matrix). Name the verifier in the skill's done-test.
- **Prefer cheap certificates.** Shape claims and outputs so that evidence of correctness is cheap to check even when producing it was not (a 2500×2500 Hadamard matrix is trivially verified; the conjecture is not — Daniel Litt, Harvard CMSA, 2026). Rank each guarantee a skill makes by certificate tier — *proved* (exhaustive algebraic check) → *exhaustively checked* (every case at a deterministic setting) → *estimated* (sampled) — state the tier with the output, and pin every verified object by hash so a later claim refers to exactly the thing that was checked.
- **The agent is a guest in the document.** In any skill that touches the user's notes, manuscripts, or code, the agent reads before writing, extends rather than duplicates, leaves the author's structure and voice intact, and makes its contributions distinguishable (comments, suggestions, marked cells) — never silent rewrites. "Guest" is the leading word; it says what the agent *is* rather than what it must not do.

- **Exhaustive completion criteria.** Every protocol ends with an explicit done-test that prevents premature completion: "done when the frontier is empty", "done when an implementer could build it without asking a single question", "done when every named gap is closed or written to the note". "Wrap up" is not a criterion.
- **Fact/decision boundary.** Finding facts is the agent's job, never the user's; decisions are the user's — put each to them and wait. Any skill that asks the user questions must respect this line.
- **Recommended-answer device** (elicitation skills only): ship every question with a veto-able recommended answer, keeping the user's load on judgment. Never use it in learning skills, where generation IS the exercise.
- **Graceful degradation.** Name external dependencies (companion skills, tools, paths) and state what to do when they are absent: apply the principle, skip the mechanics, tell the user what was skipped.

## Maintenance norms

- **Semantic versioning** in frontmatter; bump minor for content additions, patch for wording fixes. Note major state changes in the version line only when they gate usage.
- **Every content change propagates** to every location that carries the skill (working copy, backups, published copy) in the same session; published copies strip environment-specific paths and tool names.
- **Third-party skills** (installed, not authored) keep their upstream LICENSE beside them and are never republished as one's own.
- **Attribution**: when a device is adopted from someone else's skill, credit it inline ("adapted from mattpocock/skills `grilling`, MIT").

## Synthesizing skills from source corpora

(adapted from virgiliojr94/book-to-skill, MIT)

- **Extract structure, not summaries.** Capture named frameworks with their exact formulations, principles, techniques, and anti-patterns — not chapter recaps. Preserve the author's precision: "The 5 Whys" is not "ask why multiple times". A skill is a toolkit, not a book report.
- **REPL over the corpus.** Treat a large source as a queryable corpus, never a single read. `grep -n` for section offsets, `sed -n` for the slice you need, `grep -c` to confirm a framework is actually present before citing it. A 200-page book is ~75k tokens; re-reading it once per chapter costs ~2M input tokens, while targeted slices keep cost proportional to the output.
- **Layer the output.** Keep the SKILL.md body under ~4,000 tokens and front-load it (compaction truncates from the end). Push depth into on-demand files that cost nothing until loaded: per-chapter notes, `glossary.md` (terms), `patterns.md` (techniques), and `cheatsheet.md`. The cheatsheet is a reasoning aid, not a keyword list — it carries the author's *judgment*: decision rules ("when X, do Y, because Z"), decision trees, trade-off matrices, thresholds. Bare term→definition rows belong in the glossary.
- **Synthesize, never copy.** Skills derived from third-party copyrighted material stay private; publish only from your own writing, openly licensed content, or material you are authorized to redistribute.

## Verification before claims

(adapted from obra/superpowers `verification-before-completion` and `systematic-debugging`, MIT)

- **Evidence before claims, always.** No completion or correctness claim without fresh verification evidence in the same message: identify the command that proves it, run it fresh and in full, read the whole output and exit code, then state the claim *with* its evidence. "Should work", "I'm confident", "the linter passed", "the subagent reported success" are not evidence — run it. Partial checks prove nothing; "different words so the rule doesn't apply" is spirit-over-letter evasion.
- **Root cause before fixes.** Investigate (reproduce, read the full error, trace the data flow) → find a working reference and diff against it → one hypothesis, one variable, one minimal test → then fix. Symptom fixes are failure.
- **Three failed fixes → question the architecture.** Fixes that each surface a new problem elsewhere signal a wrong design, not a failed hypothesis; do not attempt a fourth without re-examining the structure.

## Done test

A skill revision is done when: the version is bumped, every location that carries the skill has the identical (or sanitized-identical) content, no meaning exists in two places, and each new sentence changes behavior versus the default.
