---
name: org-roam-notes
description: Create, link, tag, and query research notes in an org-roam v2 vault. Use when the user mentions org-roam, a note/node, backlinks, filetags, a daily/journal entry, wants a paper or web page captured into notes, or asks to save a session summary to their notes. Encodes the networked-note practices ported from kepano/obsidian-skills (MIT), translated to org-roam.
version: 1.2.0
tags: [skill, org-roam, notes, zettelkasten, emacs, knowledge-base]
---

# org-roam notes

Author and connect notes in an org-roam v2 vault. **The agent is a guest in the vault**: it reads before it writes, extends rather than duplicates, and leaves the owner's structure intact. One note = one node with an `:ID:`. Behaviors adapted from kepano/obsidian-skills (MIT): read-before-write, link-at-the-finest-stable-anchor, typed metadata, validate-after-edit.

**Configure once**: the vault directory (`org-roam-directory`) and the index db (`org-roam-db-location`, default `~/.emacs.d/org-roam.db`; Doom users: `~/.config/emacs/.local/cache/org-roam.db`). Tables: `nodes`, `links`, `tags`, `aliases`, `refs`, `citations`, `files`.

## Read before you write

Resolve the target first, then decide new-vs-extend:

1. Search titles, aliases, and tags for an existing node. **Every value in the db is an Elisp-printed string wrapped in literal double quotes — `trim(col,'"')` on read and `'"'||:val||'"'` on match, always.**
   ```sh
   sqlite3 "$ROAM_DB" "SELECT trim(id,'\"'), trim(title,'\"'), trim(file,'\"') FROM nodes WHERE title LIKE '%topic%';"
   ```
2. If a node exists, read the file and its backlinks before appending:
   ```sh
   sqlite3 "$ROAM_DB" "SELECT trim(n.title,'\"') FROM links l JOIN nodes n ON l.source=n.id WHERE l.dest='\"<uuid>\"';"
   ```
3. **Extend** the existing node when the material is the same concept, a refinement, or a result about it. **Create** a new node when it is a distinct concept other notes will want to link to on its own. When unsure, ask in one line.

## Create a node

File-level node — the drawer sits above `#+title`:

```org
:PROPERTIES:
:ID:       <uuidgen, lowercase>
:END:
#+title: Concept Name
#+filetags: :physics:qec:
```

Heading-level node — give the heading its own `:PROPERTIES:`/`:ID:` drawer; it becomes independently linkable (the org equivalent of a block reference). Add `:ROAM_ALIASES: "Alt name"` in the drawer so links resolve under several names.

## Link by ID

Internal references are ID links, never filenames: `[[id:<uuid>][Display text]]`. External URLs: `[[https://…][text]]`. Papers: `[cite:@bibtexkey]` (citar / org-cite against the user's .bib), not a bare arXiv URL. To link a sub-part of a note, give that heading its own `:ID:` and link to it.

**Link vs transclude**: link to *reference* another node (default). Transclusion (`#+transclude: [[id:<uuid>]]`) shows content live in place and requires the `org-transclusion` package — check it is installed before using it; offer it otherwise.

## Metadata and tags

Tags are file-level `#+filetags: :a:b:` (hierarchy by convention, e.g. `:qec:codes:`) or heading tags. Properties are typed by convention only — dates as `[2026-09-19]`, numbers bare, links as `[[id:…]]` — so cast on read. Guard every property read: a key may be absent on a node; treat missing as empty, never error.

Admonitions use org blocks — `#+begin_quote`, `#+begin_note`, `#+begin_warning` — chosen by *meaning*, not appearance.

## Encode the user's house rules here

A notes skill should carry the user's own formatting conventions so every note reads the same. Example (a physics user): display math is `\begin{equation}…\end{equation}` (multi-line environments nest inside), never `\[…\]`; angular-frequency quantities quoted as `X/2π` in MHz; parameter values stated directly, never attributed to a source script.

## Capture a web page or paper

Extract clean text first, then convert: `defuddle parse <url> --md` (if installed; else a plain fetch) → `pandoc -f markdown -t org` → wrap in a node with `:ID:`, `#+title`, `#+filetags`, and a source link. For a paper, the node's `:ROAM_REFS:` holds the arXiv/DOI URL so org-roam treats it as the reference note.

## Open questions as pre-commitment

Before any model is asked an open research question, the question goes into the vault first: an `* Open questions` heading in the thread's node (or a dedicated node tagged `:open:`), each entry stating the question, what would count as an answer, and how that answer would be checked cheaply. A question written down first is a commitment to *understand* its answer, not merely to receive it (Daniel Litt's problem lists, Harvard CMSA, 2026). When an answer arrives — from a model, a paper, or a computation — the owner records the resolution in their own words under the entry; the agent adds the source link and the check that was run.

## Daily notes

`org-roam-dailies` or org-journal, one file per day. Append under today's heading. If the user indexes TODO-family keywords from journal files elsewhere, use those keywords deliberately.

## Done test

- The node has an `:ID:`, a `#+title`, and at least one filetag.
- Every internal reference is an `[[id:…]]` link, and every `id:` resolves to a row in `nodes`.
- If extending, the prior content and backlinks were read first and not duplicated.
- The db reflects the change: after save (autosync) or `M-x org-roam-db-sync`, a `nodes` query returns the new node.
- No orphan: the note is reachable from at least one other node or a daily note.

## What did not transfer from Obsidian

Canvas (org-roam-ui renders the graph read-only), Bases saved views (use `org-ql` or SQL ad hoc), `cssclasses`, and the plugin-dev loop. Wikilinks' rename-tracking rationale is moot — ID links are rename-proof by construction.
