---
name: skill-authoring-conventions
description: House rules for writing and revising agent skills (SKILL.md files). Load whenever authoring a new skill, revising an existing one, or reviewing a skill's design. Distilled from Matt Pocock's writing-for-agents (mattpocock/skills, MIT) plus this collection's own norms.
version: 1.0.0
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

- **Exhaustive completion criteria.** Every protocol ends with an explicit done-test that prevents premature completion: "done when the frontier is empty", "done when an implementer could build it without asking a single question", "done when every named gap is closed or written to the note". "Wrap up" is not a criterion.
- **Fact/decision boundary.** Finding facts is the agent's job, never the user's; decisions are the user's — put each to them and wait. Any skill that asks the user questions must respect this line.
- **Recommended-answer device** (elicitation skills only): ship every question with a veto-able recommended answer, keeping the user's load on judgment. Never use it in learning skills, where generation IS the exercise.
- **Graceful degradation.** Name external dependencies (companion skills, tools, paths) and state what to do when they are absent: apply the principle, skip the mechanics, tell the user what was skipped.

## Maintenance norms

- **Semantic versioning** in frontmatter; bump minor for content additions, patch for wording fixes. Note major state changes in the version line only when they gate usage.
- **Every content change propagates** to every location that carries the skill (working copy, backups, published copy) in the same session; published copies strip environment-specific paths and tool names.
- **Third-party skills** (installed, not authored) keep their upstream LICENSE beside them and are never republished as one's own.
- **Attribution**: when a device is adopted from someone else's skill, credit it inline ("adapted from mattpocock/skills `grilling`, MIT").

## Done test

A skill revision is done when: the version is bumped, every location that carries the skill has the identical (or sanitized-identical) content, no meaning exists in two places, and each new sentence changes behavior versus the default.
