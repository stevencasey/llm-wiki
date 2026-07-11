# Ingest Pipeline Spec

The ingest pipeline is the quality gate for adding external sources to the wiki. It
enforces the 5-step procedure defined in `wiki/CLAUDE.md` and runs the lint checklist
before reporting completion.

---

## User-facing behaviour

When a user asks Claude to add an article, book, paper, podcast, or other external source:

1. Claude creates a source page in `sources/` with a summary, key points, and any
   notable quotes or data.
2. Claude updates or creates entity pages for every named person, company, tool, or place
   mentioned in the source.
3. Claude updates or creates concept pages for every significant idea introduced.
4. Claude updates `index.md`: adds the source to `## Sources`, updates `## Entities` and
   `## Concepts` entries for any pages touched.
5. Claude runs the standard lint checklist. Blockers are fixed inline without asking the
   user. Warnings are reported in the completion summary.

The user receives a plain-English completion summary: which pages were created, which were
updated, and any warnings to be aware of.

**Capture (personal notes)** is a separate, lighter flow: Claude appends to the daily note
or creates a named note, updates `index.md` only if a new daily file was created, and
offers (but does not require) to scan for promotable content.

---

## Constraints

- The 5 steps are canonical and ordered. No step may be skipped.
- Lint blockers must be resolved before the ingest is reported as complete.
- Claude must not ask the user to resolve lint failures — it fixes them inline.
- Ingest must be idempotent: ingesting the same URL twice must not create duplicate pages
  or duplicate index entries.
- The pipeline must work entirely through the Claude chat interface; no terminal access
  is required from the user.
- Input types in scope for phase 1: URLs (articles, papers), plain text, PDF files.
- Input types out of scope for phase 1: audio transcripts, images, video.
