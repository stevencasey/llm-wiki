# Wiki Operational Spec

You are reading and writing a personal knowledge base. Follow these rules exactly on every operation. Do not improvise structure, invent new directories, or skip the lint step.

---

## Directory Layout

```
wiki-root/
  CLAUDE.md          ← this file
  schema.md          ← human-readable reference
  index.md           ← master navigation index (update on every ingest)
  entities/          ← named things: people, companies, tools, places
  concepts/          ← abstract ideas, frameworks, methodologies, terms
  sources/           ← external inputs: articles, books, papers, podcasts
  synthesis/         ← cross-cutting analyses spanning 2+ sources
  notes/             ← personal capture + drop zone for files to sweep (see Sweeping)
  .wiki-ingest-ledger.json  ← processed-state ledger for the notes sweep (not a page)
```

Never create directories outside this structure. The one permitted non-page file at the
root is `.wiki-ingest-ledger.json` (see "Sweeping the Notes Folder"). Do not create any
other files at the root.

---

## Page Types

| Type | Directory | Use when |
|---|---|---|
| `entity` | `entities/` | The subject is a named, discrete thing (person, company, tool, place) |
| `concept` | `concepts/` | The subject is an abstract idea, methodology, or recurring term |
| `source` | `sources/` | The page represents an external input (article, book, paper, podcast, talk) |
| `synthesis` | `synthesis/` | The page draws conclusions across 2 or more sources |
| `note` | `notes/` | The user's own capture: a thought, meeting notes, a daily log, a fleeting idea |

If a page could be either entity or concept, ask: is it a specific named instance (entity) or a general class of thing (concept)?

Notes are the exception to most structural rules. They are capture buffers, not permanent knowledge. Content in notes should be promoted to the main taxonomy over time.

---

## Filename Rules

- Format: `kebab-case.md`, lowercase, no spaces
- Entity: `firstname-lastname.md`, `company-name.md`, `product-name.md`
- Concept: the canonical name of the idea (`zettelkasten.md`, `retrieval-augmented-generation.md`)
- Source: `author-year-keyword.md` (e.g., `karpathy-2024-llm-wiki.md`)
- Synthesis: `topic-a-vs-topic-b.md` or `topic-analysis.md`
- Note (daily): `YYYY-MM-DD.md` — date is the subject, this is the one exception to the no-dates rule
- Note (named): `topic-slug.md` (e.g., `meeting-product-roadmap.md`, `idea-search-interface.md`)

Never use dates as the primary filename component outside of `notes/`. Names elsewhere must be semantically meaningful.

---

## Required Frontmatter

Every page must open with this block. No exceptions.

```yaml
---
type: entity | concept | source | synthesis
title: Human Readable Title
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

Additional required fields by type:

**source pages** — add:
```yaml
author: Name or handle
date_published: YYYY-MM-DD
url: https://...          # or isbn: 978-...
```

**entity and concept pages that came from a source** — encouraged, not required:
```yaml
provenance: [[source-slug]]
```

**note pages** — add:
```yaml
promoted: false           # set to true once all key content has been promoted to the main taxonomy
```

---

## Cross-Link Rules

- Use `[[slug]]` wikilinks everywhere. The slug is the filename without `.md`.
- Never write a `[[slug]]` link to a page that does not exist. Create a stub first, then link.
- A stub is a page with valid frontmatter and a single `## Summary` section containing one sentence.
- Synthesis pages must include wikilinks to every source they draw from in the page body.

---

## What to Do When Capturing a Personal Note

When the user dictates a thought, shares meeting notes, or asks you to record something they wrote:

1. **Determine the note type.** Is this a daily log entry (use today's date file `YYYY-MM-DD.md`) or a named note with a distinct topic (use `topic-slug.md`)?
2. **Open or create the note file** in `notes/`. For daily notes, append to the existing file for today if it exists.
3. **Write the content** under a timestamped heading (`### HH:MM` for daily notes, or a descriptive heading for named notes).
4. **Tag the note** in frontmatter with the topics it touches.
5. **Do not update `index.md` for every note** — only update the `## Notes` section when a new daily file is created (not per-entry).
6. **Do not run the full ingest lint** — notes are exempt from orphan and provenance checks.
7. **Scan for promotable content** (optional, at the end): if the note clearly describes an entity, concept, or cross-source conclusion, offer to promote it. Do not promote without user confirmation.

**A note is ready to promote when:** the same idea has appeared in 2+ notes, or the user explicitly asks to promote it, or it contains a stable factual claim that should be permanently accessible.

**To promote a note entry:**
1. Create the appropriate entity, concept, or synthesis page in the main taxonomy.
2. Add a `provenance:` field pointing back to the note (e.g., `provenance: [[2026-07-11]]`).
3. Link from the note to the new page.
4. Set `promoted: true` in the note's frontmatter once all key content has been promoted.

---

## What to Do on Every Ingest

When a new source arrives (article, book, paper, podcast, conversation), do these steps in order:

1. **Create the source page** in `sources/` with full frontmatter and a `## Summary`, `## Key Points`, and `## Quotes or Data` section.
2. **Update or create entity pages** for every named person, company, tool, or place mentioned. Add a bullet under `## Mentions` linking back to the source.
3. **Update or create concept pages** for every significant idea introduced or developed. Add a bullet under `## Occurrences` linking back to the source.
4. **Update `index.md`** — add the source to `## Sources` (reverse-chronological), and update `## Entities` and `## Concepts` entries as needed.
5. **Run standard lint** (see below). Resolve any failures before finishing.

Do not skip step 4. The index is the LLM's primary navigation tool; an outdated index breaks all future queries.

**Ingest is idempotent.** Key every source to a stable identity (its `url`/`isbn`, or its
content hash for a dropped file). Ingesting the same source again must update the pages it
already produced — never create a second source page or duplicate entity/concept pages for
it. When re-ingesting changed content, edit the existing pages in place.

**Batch ingest does not stop to ask.** When ingest runs as one file inside a notes sweep,
fix every lint problem inline (as always) and continue to the next file — never pause to
ask the user to resolve a lint failure mid-sweep.

---

## Sweeping the Notes Folder (Auto-Ingest)

Users can drop files into `notes/` and have them picked up. This is an **on-demand sweep**,
not a background watcher: it runs only when you are asked to sweep (e.g. via the `/query`
skill's startup, a "sweep my notes" request, or at the start of a session if the user asks).
Nothing runs between sessions.

**The ledger.** `.wiki-ingest-ledger.json` at the wiki root is the sole record of what has
been processed. It is a JSON object keyed by file path; each entry stores:

```json
{
  "notes/some-article.md": {
    "hash": "<sha256 of the file's current contents>",
    "kind": "source | note",
    "pages": ["sources/author-2026-topic", "entities/some-person"]
  }
}
```

`pages` lists the slugs this file produced, so a re-ingest updates exactly those pages.

**To run a sweep:**

1. **Read the ledger** (create an empty `{}` ledger if none exists).
2. **List every file in `notes/`.** For each file, compute its content hash and compare to
   the ledger:
   - **Not in the ledger** → it is new. Process it (step 3), then add a ledger entry.
   - **In the ledger, hash differs** → it changed. Re-ingest it (step 4).
   - **In the ledger, hash matches** → skip it; report it as already processed.
   - If a file's path is gone but a ledger entry's hash matches a different current file,
     treat it as moved — update the path key, do not re-create pages.
3. **Process a new file — classify first:**
   - **External document** (an article, paper, transcript, clipped web page — something
     that came from outside): run the full ingest procedure above. The source page lands in
     `sources/`; the original file stays in `notes/` untouched. Record `kind: "source"` and
     every page slug produced.
   - **Personal note** (the user's own thought, meeting notes, a daily log): process it per
     the notes capture rules — update the `## Notes` section of `index.md` if it's a new
     daily file, scan for promotable content — but do not run the full source ingest.
     Record `kind: "note"`.
4. **Re-ingest a changed file:** re-run the same procedure, but **target the pages already
   listed in that file's ledger entry** — update them in place rather than creating new
   ones. Add any newly-mentioned entities/concepts; leave unrelated pages alone. Update the
   entry's `hash` and `pages`.
5. **Run lint** across everything the sweep created or changed, fixing inline as usual.
6. **Report** a plain-English summary: new files ingested, files re-ingested, files skipped.

**Never touch the user's files.** The sweep must not edit, rename, move, or delete any file
in `notes/`. The only thing that records "processed" is the ledger — never write a marker
into the user's file.

---

## Index.md Structure

Keep `index.md` in this exact format:

```markdown
# Wiki Index

## Entities
- [[entity-slug]] — one-line description

## Concepts
- [[concept-slug]] — one-line description

## Sources
- [[source-slug]] — Author, date_published, one-line summary (newest first)

## Synthesis
- [[synthesis-slug]] — one-line topic summary

## Notes
- [[YYYY-MM-DD]] — one-line summary of what was captured that day (7 most recent only)
```

Entries within each section are alphabetical, except Sources (reverse-chronological by `date_published`) and Notes (reverse-chronological by date, capped at 7 entries). Named notes appear in `## Notes` only if they contain content that has not yet been promoted.

---

## Lint Checklist

Run after every ingest. Fix every problem inline before reporting the ingest complete — never ask the user to fix these themselves.

1. **Broken links** — every `[[slug]]` resolves to an existing `.md` file. Create a stub if not.
2. **Missing frontmatter** — every page has all required fields for its type. Add what's missing.
3. **Orphaned pages** — every page in `entities/`, `concepts/`, and `synthesis/` has at least one inbound `[[slug]]` link. Pages in `notes/` are exempt. Add a link, or note it in your completion summary if no natural link exists yet.

Report what you fixed as: `FIXED [check-name]: path/to/file.md — what you did`.

This is one flat checklist, not a system of severities. If real use shows you need more nuance (e.g. distinguishing must-fix from nice-to-have), that's a signal to update this file — don't build the distinction preemptively.

---

## Answering a Question From the Wiki

When the user asks a question of the wiki — whether by typing `/query <question>` in
Claude Code, or just asking in chat — answer from the wiki, index-first:

1. **Read `index.md` first.** Use it to pick the pages relevant to the question.
2. **Open those pages** and read their contents.
3. **Answer from the pages,** and **cite the page slugs** you drew from so the user can
   verify and navigate.
4. **If the index has nothing relevant,** say so plainly and offer to ingest a source —
   do not fabricate an answer.

Use only generic file reads and the index. Never build a search index or use embeddings.

---

## What Not to Do

- Do not create files outside the five content directories (the one exception is the
  `.wiki-ingest-ledger.json` file at the root).
- Do not use dates in filenames.
- Do not link to pages that do not yet exist — create a stub first.
- Do not modify `CLAUDE.md` or `schema.md` during an ingest.
- Do not skip `index.md` updates.
- Do not create a synthesis page from a single source — that is a concept or source page.
- Do not ask the user to resolve lint failures — fix them inline.
- Do not edit, rename, move, or delete a user's file in `notes/` during a sweep — record
  processed state only in the ledger.
- Do not duplicate pages when re-ingesting — update the pages recorded in the ledger.
