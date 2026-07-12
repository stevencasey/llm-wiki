## Context

The wiki is Tier 1: Claude in a Claude Code (or Claude Desktop) session, using generic
filesystem access, no backend. Ingest and capture are Claude following `defaults/CLAUDE.md`
inside a chat turn. Two requested capabilities push on the edges of that model:

- **Auto-ingest** of files dropped into the wiki folder. The literal phrasing
  ("automatically ingested") implies a watcher, which Tier 1 forbids and `onboarding/spec.md`
  defers. This design keeps it in Tier 1 by making "automatic" mean *swept on invocation*.
- **`/query`** as a Claude Code skill for retrieval. Claude Code is a valid Tier 1
  environment, so a skill is admissible — but the wiki already mandates index-first
  reading, so `/query` is mostly a named entry point over existing behavior.

The full conflict list lives in `proposal.md`. This document records the technical
decisions the delta specs assume, and the alternatives.

## Goals / Non-Goals

**Goals:**
- Pick up new-or-changed files in a drop location without the user re-pasting them.
- Never mutate or delete the user's own files during the sweep.
- Re-ingest changed files idempotently — update the pages a file produced before, never
  duplicate.
- Provide one named, repeatable retrieval entry point (`/query`) that obeys index-first
  navigation and the no-embeddings constraint.

**Non-Goals:**
- A background file watcher or daemon (Tier 2, mechanism undecided — out of scope).
- Any parsing backend for specific file types beyond what Claude can already read in a
  session.
- Any search index, BM25, or vector embeddings (BM25 remains deferred; vectors are banned
  at every tier).
- Claude Desktop parity for `/query` (skills are Claude-Code-only; Desktop users keep the
  existing "ask Claude to answer from the wiki" chat path).

## Decisions

### D1 — "Automatic" = on-demand sweep, not a watcher
The sweep runs when invoked inside a Claude Code session (its own skill, and/or folded
into `/query`'s startup, and/or offered at session start). No process runs between
sessions. **Why:** keeps the capability inside Tier 1's "Claude + file tools, no backend"
envelope. **Alternative considered:** an OS file watcher / MCP watch server — rejected as
Tier 2 with an undecided mechanism; building it now would violate the constitution.

### D2 — External ledger for processed state, not in-file markers
Processed state is a single ledger file at the wiki root (e.g. `.wiki-ingest-ledger.json`)
mapping each source file's path → content hash → the wiki pages that ingest produced from
it. **Why:** the requirement says do not edit the user's files; an in-file `ingested:`
marker would edit them. The ledger also gives re-ingest the file→pages map it needs to
update in place. **Alternative considered:** frontmatter marker (`ingested: true`, hash) —
simpler and self-contained, but edits the user's file, so rejected unless the user prefers
it (see proposal conflict #3).

### D3 — Change detection by content hash
A file is "new" if its path is absent from the ledger, "changed" if present with a
different content hash, "unchanged" (skipped) if the hash matches. **Why:** hashing is
deterministic and mtime is unreliable across syncs/copies. **Alternative:** mtime — cheaper
but produces false re-ingests after copies/restores.

### D4 — Re-ingest updates in place via the ledger's page map
On a changed file, ingest re-runs but *targets the pages recorded in the ledger* for that
file rather than creating fresh ones, then reconciles (adds newly-mentioned entities/
concepts, leaves unrelated pages alone). **Why:** satisfies "changes updated, do not
duplicate." This makes the existing ingest spec's "no duplicate on re-ingest" line a
testable, keyed requirement (see the `ingest` delta).

### D5 — Classify each dropped file: source vs. note
The sweep does not blindly "ingest" everything, because `notes/` is capture, not ingest.
For each file it decides: an external document → run the full ingest procedure (source page
+ entities/concepts/index/lint); a personal note → capture per the notes rules. **Why:**
respects the ingest/capture split in the product language. **Open:** the drop location
itself — see Q1.

### D6 — `/query` is index-first and tool-only
`/query <question>` reads `index.md`, selects the relevant pages, reads them, and answers
with citations to page slugs. It uses only generic list/read/grep. **Why:** matches the
existing `wiki/spec.md` index-first rule and the no-embeddings / BM25-deferred constraints.
It is a *named packaging* of that behavior, plus an explicit answer-with-citations contract.

## Risks / Trade-offs

- **[Ledger drift]** If the user renames or moves a source file, the ledger's path key goes
  stale and the file looks new (re-ingested as a duplicate source). → Mitigation: key the
  ledger on content hash as a secondary identity so a moved-but-unchanged file is
  recognized; document rename handling in the sweep procedure.
- **[Scope creep toward a daemon]** Users may expect true background ingest. → Mitigation:
  the spec states plainly this is on-demand; a watcher is Tier 2, flagged in the proposal.
- **[`/query` overlaps existing behavior]** It may add ceremony without new capability. →
  Mitigation: keep it a thin skill; its only new contract is answer-with-citations. If a
  review finds it redundant, fold it into `mcp/spec.md` instead of a new spec (see Q3).
- **[Ledger file vs. schema]** A root-level ledger sits next to "five directories are
  fixed." → Mitigation: it is a file, not a directory, and is not a content page; confirm
  with the user (proposal conflict #4).

## Open Questions — RESOLVED

- **Q1 — Drop location → `notes/`.** The swept location is `notes/`, matching the
  original request ("the notes folder") and avoiding a new directory, which also settles
  conflict #4. Each file in `notes/` is classified (D5): an external document is ingested
  into `sources/`; a personal note is processed per the notes rules. The user's file stays
  in `notes/` untouched; any derived source page is a separate file.
- **Q2 — Marker → external ledger.** A single hidden ledger file
  `.wiki-ingest-ledger.json` at the wiki root. User files are never edited.
- **Q3 — `/query` → its own `query` spec, Claude-Code-only.** A Claude-Code-only subset of
  Tier 1 is accepted; Claude Desktop users keep the existing chat path (ask Claude, which
  already reads `index.md` first per `wiki/spec.md`).
- **Q4 — Watcher intent → on-demand sweep confirmed.** No background watcher; a true
  watcher would be a separate Tier 2 change.
