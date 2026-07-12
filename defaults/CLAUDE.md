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
  notes/             ← personal capture: daily notes, meeting notes, fleeting thoughts
```

Never create directories outside this structure.

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

## What Not to Do

- Do not create files outside the five content directories.
- Do not use dates in filenames.
- Do not link to pages that do not yet exist — create a stub first.
- Do not modify `CLAUDE.md` or `schema.md` during an ingest.
- Do not skip `index.md` updates.
- Do not create a synthesis page from a single source — that is a concept or source page.
- Do not ask the user to resolve lint failures — fix them inline.
