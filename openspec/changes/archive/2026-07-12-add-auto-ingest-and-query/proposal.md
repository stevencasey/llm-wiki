## Why

Today every source enters the wiki by the user pasting it into chat, and every wiki
query relies on the user remembering to ask Claude to read `index.md` first. Two gaps
follow: (1) files a user drops into their wiki folder outside a chat turn are never
picked up, and (2) there is no named, repeatable entry point for asking the wiki a
question. This change adds an on-demand ingest sweep for dropped files and a `/query`
skill for retrieval — both within Tier 1 (Claude Code + filesystem, no backend).

## What Changes

- Add an **on-demand ingest sweep**: when invoked inside a Claude Code session, Claude
  scans a designated drop location, ingests every file that is new or has changed since
  it was last processed, and skips files already processed unchanged.
- The sweep **never edits or removes the user's own files**. Processed state lives in a
  separate ledger, not in the file.
- A file whose content has **changed since last ingest is re-ingested**, updating the
  pages it previously produced **in place** — no duplicate pages.
- Add a **`/query` skill** for Claude Code that answers a user's question from the wiki,
  reading `index.md` first, using only generic file tools (no embeddings, no search
  index).
- Extend the existing **ingest** contract so re-ingest is keyed to a stable source
  identity (making "no duplicate on re-ingest" a testable requirement) and so ingest can
  run non-interactively as one item in a batch.

**Not a daemon / not a file watcher.** "Automatic" here means "swept on invocation," not
a background process. A continuously-watching inbox is Tier 2 and its mechanism is
undecided — see Conflicts below and `openspec/specs/onboarding/spec.md`.

## Capabilities

### New Capabilities
- `auto-ingest`: An on-demand sweep that ingests new-or-changed files from a drop
  location, tracks per-file processed state in an external ledger, leaves user files
  untouched, and re-ingests changed files idempotently.
- `query`: A `/query` Claude Code skill that answers a user request from the wiki,
  index-first, with generic file tools only.

### Modified Capabilities
- None. The idempotency and non-interactive-batch behaviour that would otherwise modify
  the `ingest` spec are folded into the new `auto-ingest` capability (where the sweep
  exercises them) and into the operational `defaults/CLAUDE.md`. The existing prose
  `ingest/spec.md` is not in the canonical `## Requirements` format the archiver rebuilds,
  so a delta against it is deferred rather than reformatting that spec here.

## Impact

- New specs: `openspec/specs/auto-ingest/spec.md`, `openspec/specs/query/spec.md`
  (created via this change's delta specs, merged on archive).
- `defaults/CLAUDE.md` will need operational procedures for the sweep and the ledger, and
  a `/query` skill will need to be added under the user's Claude Code config — both are
  implementation follow-ups (tasks), not part of the spec contract itself.
- Introduces one new artifact in the wiki root: an ingest **ledger** file. This touches
  the "five content directories are fixed" constraint — see Conflicts.

## Conflicts to resolve

These requirements collide with existing specs / the constitution. Each needs a user
decision before implementation; the proposed resolution is what the delta specs assume.

1. **"Automatically ingested" vs. Tier 1 "no backend, no daemon."** The constitution
   bans any custom backend/watcher in Tier 1, and `onboarding/spec.md` explicitly
   *defers* the "drop-folder (inbox) capture path," treating chat-based capture as
   sufficient until Tier 1 use proves otherwise. **Proposed resolution:** implement as an
   on-demand sweep triggered from within a Claude Code session (no always-on watcher),
   keeping it in Tier 1. **If you actually want a background watcher, this is Tier 2 and
   its mechanism is still undecided** — the sweep specs would not apply as written.

2. **"Ingest the notes folder" vs. `notes/` = capture, not ingest.** Product language
   reserves *ingest* for external sources and *capture* for personal notes, and the
   ingest spec's capture path deliberately skips the source/entity/concept/index steps.
   Fully ingesting everything dropped into `notes/` contradicts that. **Proposed
   resolution:** the sweep watches a drop location and *classifies* each file — an
   external document is ingested into `sources/` (+ entities/concepts/index/lint); a
   personal note is captured per the notes rules. **Open question:** is the drop location
   `notes/`, or a new dedicated inbox? A new top-level directory conflicts with "the five
   content directories are fixed" (see #4).

3. **"Do not edit the files" vs. "mark each as processed."** Marking a file as processed
   by writing frontmatter *is* editing it. **Proposed resolution:** store processed state
   (path + content hash + derived-page map) in a separate **ledger** file, leaving user
   files byte-for-byte untouched. **Alternative** you may prefer: a `ingested:` marker in
   the file's frontmatter — simpler, but it edits the file. The specs assume the ledger.

4. **A ledger file vs. "the five content directories are fixed."** The schema forbids new
   *directories* in the wiki root; a single ledger *file* at the root is not strictly
   forbidden but is adjacent to that rule. **Needs confirmation** that a root-level ledger
   file (and, if chosen in #2, a new inbox directory) is acceptable.

5. **`/query` vs. the existing "read `index.md` first" instruction + deferred search.**
   `wiki/spec.md` already mandates index-first reading on *any* wiki query as a
   `CLAUDE.md` instruction, and `mcp/spec.md` *defers* a BM25 search primitive. So
   `/query` is largely a **named packaging** of behavior that already exists, not new
   retrieval capability. **Open questions:** (a) Is a slash-command skill worth adding if
   it only wraps existing behavior? (b) `/query` is Claude-Code-only; Claude Desktop users
   are also Tier 1 but cannot run skills — is a Claude-Code-only subset acceptable? (c)
   Confirm `/query` must **not** introduce any search index or embeddings (deferred).
