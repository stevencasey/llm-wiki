# Ingest Spec (Tier 1)

The ingest procedure is what Claude follows — inside the chat session, using normal
file tools, no separate backend — when the user shares an external source. It's defined
operationally in `defaults/CLAUDE.md`; this spec describes the contract that procedure
must satisfy.

---

## User-facing behaviour

When a user shares an article, book, paper, podcast, or other external source (a URL,
pasted text, or a file shared in chat):

1. Claude creates a source page in `sources/` with a summary, key points, and any
   notable quotes or data.
2. Claude updates or creates entity pages for every named person, company, tool, or
   place mentioned in the source.
3. Claude updates or creates concept pages for every significant idea introduced.
4. Claude updates `index.md`.
5. Claude runs the lint check (see `wiki/spec.md`) and fixes anything broken inline.

The user gets a plain-English completion summary: what was created, what was updated.

**Capture** (a personal thought or note) uses the same mechanism, lighter weight:
Claude appends to today's note or creates a named note, and skips steps 2–4 unless the
content clearly belongs in the permanent taxonomy already.

---

## Constraints

- The 5 steps are ordered; none may be skipped.
- Claude fixes lint problems inline — it doesn't ask the user to.
- Ingesting the same source twice must not create duplicate pages.
- What Claude can ingest (a URL, a PDF, pasted text, an image) is bounded by what
  Claude's chat interface can read in a given conversation. This spec doesn't define
  separate parsing infrastructure for any file type — that would be a Tier-1-violating
  custom backend.
