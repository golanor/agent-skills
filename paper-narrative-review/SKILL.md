---
name: paper-narrative-review
description: Critique the STORY a paper's figures tell, as a handling-editor verdict only — never draws figures, never writes prose. Use when a physics paper draft (abstract + figure deck/captions) needs a narrative pass: does Fig. 1 hook, is each panel in the right figure, what analysis is missing, what should be demoted or cut. Triggers — figure story, narrative arc, is Figure 1 a hook, figure review, paper narrative, which figure, missing panel, kill list.
version: 1.0.0
license: Apache-2.0 (adapted from HughYau/AcademicForge `paper-narrative`)
tags: [skill, writing, figures, physics, critique, anti-deskilling]
---

# Paper Narrative Review

Adapted from HughYau/AcademicForge `paper-narrative` (Apache-2.0); Claude-Science plumbing (kernel, figure-composer handoff, OpenAlex) removed.

Judge the *story* a paper's figures tell and return an editor's verdict. You critique; the user draws every figure, runs every analysis, and makes every call. You name what to run and what to move — you never produce the figure or the prose. Companion to `english-writing-coach` (which owns sentence-level critique) — this skill owns the figure-level argument.

## When to use

The user has a draft (or an abstract) and a figure deck and wants to know: would Figure 1 make an editor send this out for review? Is each panel in the right figure? What analysis is missing? What should be demoted or cut? Run it *before* figure polishing, because the arc it returns tells the user which figures matter.

## Boundaries

- **You do:** read the manuscript and captions the user supplies and return the structured verdict below — hook test, narrative arc, panel moves, missing analyses to *run*, kill list, and the boldest defensible Fig. 1 claim.
- **You never:** draw, compose, or restyle a figure; write pitch, caption, or abstract prose for the user; decide on the user's behalf which analysis to run. You name the analysis; the user runs it and judges the result.

Critique what the manuscript actually says, not what you assume it means.

## Procedure

1. **Derive the brief, then hand it to the user to confirm.** From the abstract and per-figure captions, state your reading of the paper's *pitch* (the one grandest supportable claim, not the method), its *vision* (what a reader can now do), its *audience*, and its *most arresting asset* (the one image for a poster). These are your reading; the user corrects them — the pitch is the user's claim to make.
2. **Play the handling editor over the full deck.** One editorial pass across every figure. Judge STORY, not craft (readability, color, fonts are out of scope). Be opinionated — the user wants a partner, not a grader.
3. **Emit the verdict as JSON matching the schema below.** Schema only; no narration around it.
4. **Hand back, do not act.** `missing_panels` names analyses for the *user* to run against their own data; `figure_moves` and `kill_list` are proposals the user accepts or rejects. Re-run step 2 on the revised deck when asked.

## Output schema (exact)

```json
{
  "hook_verdict": {
    "would_send_for_review": "yes | weak | no",
    "why": "string",
    "fig1_is": "what Fig 1 currently claims",
    "fig1_should_be": "string"
  },
  "figure_moves": [
    {"what": "panel", "from_fig": "n", "to_fig": "m", "why": "string"}
  ],
  "missing_panels": [
    {"target_fig": "n", "what_to_show": "string",
     "analysis_needed": "the analysis the USER runs", "data_hint": "string"}
  ],
  "kill_list": [
    {"what": "string", "why": "string", "demote_to": "supplement | caption | delete"}
  ],
  "arc": [
    {"fig": "n", "role": "hook | mechanism | evidence | application | supplement", "one_line": "string"}
  ],
  "boldest_defensible_fig1": "the strongest Fig 1 claim the data still support"
}
```

## Done test

The pass is complete when the JSON validates against the schema, every `missing_panels` entry names a concrete physics quantity to compute (a threshold fit, a decay extraction, a limit test — not "add more data"), every `kill_list` entry has a `demote_to` target, the `arc` covers the intended main figures in reading order, and the verdict has been handed back with zero figures drawn and zero sentences of the user's prose written. Converged when `would_send_for_review == "yes"` and both `figure_moves` and `missing_panels` are empty.
