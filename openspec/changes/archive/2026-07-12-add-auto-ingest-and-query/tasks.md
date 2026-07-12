## 1. Resolve conflicts before implementing

- [x] 1.1 Confirm "automatic" = on-demand sweep, not a background watcher (proposal conflict #1 / design Q4)
- [x] 1.2 Decide the drop location: `notes/` vs. a new dedicated inbox directory (conflict #2 / Q1) — resolved to `notes/`
- [x] 1.3 Decide processed-state marker: external ledger vs. in-file frontmatter (conflict #3 / Q2) — resolved to external ledger
- [x] 1.4 Confirm a root-level ledger file is acceptable vs. the fixed-five-directories rule (conflict #4) — one hidden ledger file permitted, no new directory
- [x] 1.5 Decide `/query`'s home and accept the Claude-Code-only subset of Tier 1 (conflict #5 / Q3) — own `query` spec, Claude-Code-only

## 2. Auto-ingest procedure in defaults/CLAUDE.md

- [x] 2.1 Document the ledger format (path, content hash, kind, derived-page slugs) and its location
- [x] 2.2 Write the sweep procedure: enumerate `notes/`, hash each file, classify new/changed/unchanged against the ledger
- [x] 2.3 Write the classify step (external document → ingest to `sources/`; personal note → capture per notes rules)
- [x] 2.4 Write the re-ingest-in-place step: update pages recorded in the ledger for a changed file, no duplicates
- [x] 2.5 Add the read-only guarantee: the sweep never edits, renames, moves, or deletes user files
- [x] 2.6 Add moved-but-unchanged handling (recognize by content hash, do not duplicate)

## 3. Ingest idempotency and batch behaviour

- [x] 3.1 Update the ingest procedure so re-ingest is keyed to stable source identity and updates existing pages
- [x] 3.2 Confirm batch ingest fixes lint inline without prompting, so a multi-file sweep does not stall

## 4. /query skill

- [x] 4.1 Add a `/query` Claude Code skill (defaults/skills/query/SKILL.md) that reads `index.md` first, selects and reads relevant pages, and answers
- [x] 4.2 Make the answer cite the page slugs it used
- [x] 4.3 Handle the no-relevant-page case in plain language, optionally suggesting an ingest
- [x] 4.4 Verify the skill uses only generic file tools — no search index, no embeddings
- [x] 4.5 Document the chat answer path in defaults/CLAUDE.md so Claude Desktop users are covered

## 5. Validate and verify by real use

- [x] 5.1 Run `openspec validate add-auto-ingest-and-query` and fix any schema errors
- [ ] 5.2 Dry-run the sweep on a folder with a new file, an unchanged file, and an edited file; confirm no duplicates and files untouched
- [ ] 5.3 Dry-run `/query` against a small wiki and confirm index-first reads and citations
