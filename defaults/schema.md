# Wiki Schema Reference

This document explains the design of the wiki — what the rules are and why they exist. It is the human-facing companion to `CLAUDE.md`. When you want to evolve the schema, change this document first and then update `CLAUDE.md` to match.

---

## Philosophy

This wiki is designed to **compound**. Each new source you add should make existing pages richer, not just create a new isolated note. The schema enforces this by requiring that every ingest touch entity pages, concept pages, and the index — not just create a new source page. That propagation is what turns a pile of notes into a navigable knowledge base.

The LLM is the primary reader and writer, but the structure should also be human-readable. These goals are mostly aligned: clear naming, consistent frontmatter, and a maintained index serve both.

---

## Directory Structure

```
wiki-root/
  CLAUDE.md          ← agent operational spec
  schema.md          ← this document
  index.md           ← master navigation index
  entities/          ← named things
  concepts/          ← abstract ideas
  sources/           ← external inputs
  synthesis/         ← cross-cutting conclusions
  notes/             ← personal capture (daily notes, meetings, fleeting thoughts)
```

Five content directories. No others. Four hold the permanent, structured knowledge base. The fifth (`notes/`) is the capture buffer — content lives there temporarily and is promoted into the other four over time. A proliferating directory tree beyond these five is a sign the taxonomy is breaking down.

---

## Page Types

### Entity (`entities/`)

Use for any named, discrete thing: a person, company, product, tool, or place. Entities are nouns you refer to repeatedly. When a new source mentions Andrej Karpathy, you update `entities/andrej-karpathy.md` — you do not create a new page per mention.

Entity pages accumulate mentions over time. They become increasingly useful as a single place to see everything the wiki knows about a subject.

### Concept (`concepts/`)

Use for abstract ideas, frameworks, methodologies, and recurring terms. The test: would a textbook have an entry for this? If yes, it is a concept. Examples: `retrieval-augmented-generation.md`, `spaced-repetition.md`, `zettelkasten.md`.

Concepts also accumulate. A concept page after ten sources is richer than after one — it starts showing where different authors agree and disagree.

### Source (`sources/`)

Use for every external input you ingest: articles, books, papers, podcasts, talks, conversations. The source page is the provenance layer. It is how you answer "where did this come from?" for any claim in the wiki.

Source pages do not grow after creation (except for a `## Notes` section you may add later). Their job is to record what the external input said, not to synthesize across inputs.

### Synthesis (`synthesis/`)

Use when you have something to say that requires drawing on two or more sources. A synthesis page is your analysis, not a summary of one thing. It must include wikilinks to every source it draws from in the page body.

Do not create a synthesis page from one source — that is just a long source summary, and it belongs in the source page's `## Key Points` section.

### Note (`notes/`)

Use for anything you generate yourself: a thought you had, notes from a meeting, something you want to remember before you know where it belongs. Notes are the entry point for personal knowledge, not external sources.

Notes are explicitly temporary. They are a capture buffer, not a destination. Content in notes should eventually be promoted — moved or copied into the permanent taxonomy as entities, concepts, or synthesis pages once it's stable enough to deserve a permanent home. A note that never gets promoted is still valuable as a searchable record, but the goal is to move the important parts forward.

**Two note formats:**

*Daily notes* (`notes/YYYY-MM-DD.md`) — a single file per day, append-only. Use for quick captures during the day: a passing thought, a link with a reaction, something you heard. Each entry gets a timestamped heading (`### HH:MM`). This is the only file type in the wiki where the date is the filename.

*Named notes* (`notes/topic-slug.md`) — for captures with a clear subject that deserves its own file: meeting notes, a worked-through idea, a draft synthesis. Use when you know at capture time that this stands on its own.

---

## Naming Convention

**Why kebab-case?** It is machine-friendly (no quoting, no encoding issues in wikilinks) and human-readable.

**Why no dates in filenames (except notes)?** A file named `2024-03-15-article.md` says nothing about what the file contains. A file named `karpathy-2024-llm-wiki.md` is navigable and searchable. The `created` frontmatter field records the date; the filename records the subject. The exception is daily notes — there the date is the subject, so `2026-07-11.md` is the correct and only meaningful name for that file.

**Why author-year-keyword for sources?** It mirrors academic citation convention, which is a solved problem. It disambiguates two articles about the same topic, and the year helps without making the year the primary key.

**Filename examples:**
- `andrej-karpathy.md` (entity — person)
- `nvidia.md` (entity — company)
- `obsidian.md` (entity — tool)
- `retrieval-augmented-generation.md` (concept)
- `spaced-repetition.md` (concept)
- `karpathy-2024-llm-wiki.md` (source)
- `attention-mechanism-analysis.md` (synthesis)
- `2026-07-11.md` (daily note)
- `meeting-product-roadmap.md` (named note)

---

## Frontmatter

### Required fields (all pages)

```yaml
---
type: entity | concept | source | synthesis | note
title: Human Readable Title
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

**`type`** — Controls which lint rules apply and how the index categorizes the page.

**`title`** — The human-readable name. Can include punctuation and mixed case. This is what appears in the index.

**`tags`** — The taxonomy. Keep tags lowercase and consistent. Tags are how the deep lint contradiction check finds related pages. If you tag loosely, you get too many false positives in the contradiction check. If you tag too narrowly, contradictions slip through.

**`created` / `updated`** — ISO 8601 date. `updated` must be changed whenever the page content changes.

### Source-specific fields

```yaml
author: Name or handle
date_published: YYYY-MM-DD
url: https://...
```

Or `isbn:` instead of `url:` for books. `date_published` is when the source was published, not when you ingested it. `created` records ingestion date.

### Provenance field (entity and concept pages)

```yaml
provenance: [[source-slug]]
```

This field answers "how did this page come to exist?" It's encouraged, not required — add it when convenient, but the lint check does not enforce it. If it turns out you rely on this for navigation once the wiki grows, that's a signal to make it a required field later; don't enforce it before you know you need it.

### Note-specific field

```yaml
promoted: false
```

Set to `true` once all key content in the note has been promoted into the permanent taxonomy. A `promoted: true` note stays in the wiki as a record but no longer surfaces in the `## Notes` index section. Notes with `promoted: false` that are older than 30 days should be reviewed.

---

## The Index

`index.md` is the most important file in the wiki for navigation. The LLM reads the index before reading anything else. An outdated index is worse than no index — it actively misdirects.

The index must be updated on every ingest. This is not optional.

Structure:

```markdown
# Wiki Index

## Entities
- [[andrej-karpathy]] — AI researcher, founder of Eureka Labs
- [[nvidia]] — GPU manufacturer, dominant in AI compute

## Concepts
- [[retrieval-augmented-generation]] — grounding LLM outputs with retrieved documents
- [[spaced-repetition]] — memory technique using increasing review intervals

## Sources
- [[karpathy-2024-llm-wiki]] — Karpathy, 2024-11, overview of LLM wiki pattern
- [[voss-2011-fast-reading]] — Voss, 2011-09, techniques for rapid nonfiction reading

## Synthesis
- [[llm-wiki-vs-rag-analysis]] — comparison of wiki-first vs RAG retrieval architectures

## Notes
- [[2026-07-11]] — product roadmap discussion, idea for search interface
- [[2026-07-10]] — reading notes on attention mechanisms
```

The one-line descriptions are critical — they are what the LLM uses to decide which file to open. Write them to answer "what is this about?" not "what type of thing is this?"

---

## Cross-Links

Use `[[slug]]` wikilinks throughout. The slug is the filename without `.md`.

**The no-orphan rule:** every content page must be reachable from at least one other page via a wikilink. The index counts. A page reachable only from the index is acceptable but weak — it means no other content has referenced it yet.

**The no-broken-link rule:** never write a link to a page that does not exist. Create a stub first. A stub has valid frontmatter and one sentence in a `## Summary` section. Stubs are fine; broken links are not.

**Synthesis pages** must include wikilinks to every source they draw from in the page body — this is how you trace a conclusion back to its evidence. There is no separate frontmatter field for this; the body links are sufficient and less maintenance.

---

## Lint

The lint step is the ingestion quality gate. It runs after every ingest, and Claude
fixes what it finds before the ingest is considered complete.

| Check | Applies to | What it catches |
|---|---|---|
| Broken link check | all | `[[slug]]` references to missing files |
| Missing frontmatter | all | Pages lacking required fields |
| Orphan check | entities, concepts, synthesis | Pages no other page links to |

Notes are exempt from the orphan check — they are the origin of content, not a
reference target.

This is deliberately one flat list rather than a severity taxonomy (blockers vs.
warnings) or a contradiction-detection system. Both of those are real ideas worth
having eventually — but only once real use has shown the flat version isn't enough.
Building that structure in before a single page exists is exactly the kind of premature
process this schema is trying to avoid. If you find yourself wanting it, that's the
signal to add it, not a reason to have built it up front.

**When a lint problem fires**, Claude fixes it inline — creates the missing stub, adds
the missing frontmatter, adds the missing index entry — and notes what was fixed in the
completion summary. The user does not need to intervene.

---

## Promotion

Promotion is the process of moving content from `notes/` into the permanent taxonomy. It is the mechanism by which personal capture becomes structured, reusable knowledge.

**When to promote:**
- The same idea has appeared in 2 or more notes independently
- The user explicitly asks to promote something
- A note entry contains a stable factual claim that should be permanently accessible
- A note describes a named person, company, or tool you expect to reference again

**How to promote:**
1. Identify the content in the note that belongs in the permanent taxonomy.
2. Determine the right page type (entity, concept, or synthesis).
3. Create the page in the appropriate directory with full frontmatter. Set `provenance:` to the note's slug.
4. Add a wikilink from the note to the new page so the note's context is preserved.
5. Once all key content from a note has been promoted, set `promoted: true` in the note's frontmatter and remove it from the `## Notes` index section.

**What not to promote:** Passing reactions, one-off reminders, time-sensitive context ("need to follow up with X before Friday"). These are fine as notes; they don't need a permanent home.

---

## Evolving the Schema

`schema.md` is the authority. When you want to change a rule, update this file first to record the decision and the reason, then update `CLAUDE.md` to reflect it operationally. Any structural change (new directory, renamed page type, new required field) requires a lint pass afterward to verify no existing pages broke.

---

## Worked Example: Ingesting an Article

**Article:** "Beyond RAG: Karpathy's Compounding Knowledge" (Level Up Coding, 2024)

**Step 1 — Create the source page:**

File: `sources/karpathy-2024-llm-wiki.md`
```yaml
---
type: source
title: Beyond RAG — Karpathy's Compounding Knowledge
tags: [llm, knowledge-management, wiki]
created: 2026-07-11
updated: 2026-07-11
author: Level Up Coding
date_published: 2024-11-01
url: https://levelup.gitconnected.com/beyond-rag-...
---

## Summary
Overview of Andrej Karpathy's LLM wiki pattern and how it differs from RAG.

## Key Points
- The wiki pattern compounds knowledge; RAG retrieves but does not accumulate
- A CLAUDE.md / schema.md is required to keep the wiki coherent at scale
- Index maintenance is the operational bottleneck

## Quotes or Data
> "The index is the substitute for vector search in this architecture."
```

**Step 2 — Update the entity page for Karpathy:**

File: `entities/andrej-karpathy.md` (create if new, update if exists)
Under `## Mentions`, add:
```
- [[karpathy-2024-llm-wiki]] — described his LLM wiki pattern
```

**Step 3 — Update concept pages:**

- `concepts/retrieval-augmented-generation.md` — add mention under `## Occurrences`
- `concepts/knowledge-management.md` — create if new, with provenance pointing to this source

**Step 4 — Update `index.md`:**

Add to `## Sources` (at the top, newest first):
```
- [[karpathy-2024-llm-wiki]] — Level Up Coding, 2024-11, overview of LLM wiki pattern
```

Add or update `## Entities` entry for Karpathy.
Add or update `## Concepts` entries for any new concept pages created.

**Step 5 — Run lint.** All four standard checks must pass.
