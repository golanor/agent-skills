# Agent Skills for Learning, Writing, and Deep Work

A set of four **agent skills** — plain-text `SKILL.md` files — that turn a coding
or research assistant from an *answer machine* into a *study partner*. Each one
encodes a deliberate, high-friction workflow designed to prevent the core
failure mode of AI help: **deskilling**, where the tool does the thinking and
you learn nothing.

They are tool-agnostic markdown. Drop them into whatever agent reads skill files
(Claude Code, Cursor, Kiro, Windsurf, or your own harness) and reference them by
name or trigger.

## The five skills

| Skill | What it does | Core principle |
|---|---|---|
| [`socratic-chavrusa`](socratic-chavrusa/SKILL.md) | A Socratic / *chavrusa* learning partner that argues, questions, and withholds the answer until you've done the cognitive work. | A tunable friction dial (L0–L6): the agent withholds scaffolding so *you* retrieve, generate, and self-explain first. |
| [`english-writing-coach`](english-writing-coach/SKILL.md) | Coaches your prose against explicit rubrics distilled from 13 craft books — it diagnoses and names problems, but **you** write every sentence. | Never ghost-writes. Every critique cites a named rubric item so you acquire the vocabulary, not just the corrected text. |
| [`anki-forge`](anki-forge/SKILL.md) | Coaches you through authoring high-quality spaced-repetition cards. **You** draft every card; the agent critiques it against a rubric. | Card-writing *is* the studying (generation effect). The agent critiques and rejects; it never drafts cards for you. |
| [`stage-lessons`](stage-lessons/SKILL.md) | Durable, in-repo project memory for long multi-stage work — per-stage reports and lessons files so a later session inherits calibration decisions and retracted conclusions instead of re-breaking them. | Write conclusions down *during* the run; revise by appending, never rewriting; read before you write. |
| [`skill-authoring-conventions`](skill-authoring-conventions/SKILL.md) | House rules for writing and revising skills like these — structure, language, behavior design, maintenance. | Single source of truth, positive phrasing, leading words, and an explicit done-test on every protocol. Distilled from Matt Pocock's `writing-for-agents`. |

## How they fit together

The skills cross-reference each other, but each also works standalone (if a
companion is absent, apply the principle and skip the mechanics).

```
        socratic-chavrusa   ← the friction engine (L0–L6 dial, chavrusa stance)
          │        │   │
          │        │   └──────────────► stage-lessons
          │        │        (comprehension gate at each stage boundary;
          │        │         "Agent-Written Simulations" model-card protocol)
          │        │
          │        └──────────────────► english-writing-coach
          │             ("Writing coach" mode delegates the full rubrics here;
          │              W0–W5 ladder mirrors the L0–L6 dial)
          │
          └───────────────────────────► anki-forge
                ("Anki card forge" & "Leech triage" modes delegate the
                 card rubric, storage, and leech mechanics here)
```

`socratic-chavrusa` is the hub: the writing coach and card forge are specialized
delegates of its coaching modes, and `stage-lessons` invokes its anti-deskilling
rules as the comprehension gate on agent-written code.

## Installing into an agent

These are ordinary markdown files with YAML frontmatter (`name`, `description`,
`triggers`). Placement depends on your tool:

- **Claude Code** — copy each folder into `.claude/skills/` (project) or
  `~/.claude/skills/` (global).
- **Cursor / Windsurf** — point your rules/skills directory at these files, or
  paste the relevant one into a rule.
- **Kiro** — copy each folder into your skills directory.
- **Any custom harness** — load the `SKILL.md` body into context when its
  trigger keywords appear, or just `@`-mention the file.

A minimal manual use needs no installation at all: paste a skill's body into the
chat and say "follow this."

## Design notes

- **Friction is front-loaded, not permanent.** Every skill closes the loop with
  the complete, correct answer *after* you've attempted it. "Just tell me" is a
  standing, instant override in all of them.
- **The user is the author.** The writing coach doesn't rewrite your sentences;
  the card forge doesn't draft your cards; the stage-lessons gate has *you*
  write the durable conclusions. The agent critiques, names principles, and
  points at the neighborhood of the problem — never the fix.
- **Tool-agnostic on purpose.** These were extracted from a personal setup and
  scrubbed of any dependency on a specific notes app, scheduler, or environment.
  Where a skill mentions a concrete tool (AnkiConnect, org-roam, Obsidian) it is
  as *one example among alternatives*, never a requirement.

## Attribution

These skills **distill** ideas from published work; the principles belong to
their authors, the phrasing and the workflow synthesis are mine.

- **Learning science:** Bjork & Bjork (New Theory of Disuse), Kornell & Bjork,
  Kintsch (Construction-Integration), Andy Matuschak, Michael Nielsen, LeanAnki,
  Piotr Woźniak (SuperMemo), zettelkasten.de.
- **Skill design:** Matt Pocock's [`writing-for-agents` and `grilling`](https://github.com/mattpocock/skills)
  (MIT) — the authoring conventions and the frontier-elicitation device in the
  Research-Design Grill mode are adapted from there.
- **Writing craft:** Zinsser, Strunk & White, Lamott, Pinker, Dreyer, Le Guin,
  Truss, Williams & Bizup, Bell, Turabian, Booth/Colomb/Williams, Silvia, and
  *Good Writing: 36 Ways to Improve Your Sentences*.

If you build on these, keeping the attribution and pointing readers at the
primary sources is appreciated.

## License

[MIT](LICENSE) — use, modify, and redistribute freely. The license covers these
files; it does not extend to the copyrighted works they cite.
