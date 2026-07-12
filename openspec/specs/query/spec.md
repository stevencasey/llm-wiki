# query Specification

## Purpose

Tier 1 (Claude Code only). Give the user one named entry point — `/query <question>` — for
asking the wiki a question. It formalises the index-first retrieval already mandated by
`wiki/spec.md`: read `index.md`, open the relevant pages, answer from their contents, and
cite the page slugs used. It uses generic file tools only — no search index, no embeddings
(BM25 remains deferred per `mcp/spec.md`). Claude Desktop users are also Tier 1 but cannot
run skills; they keep the equivalent chat path (ask Claude, which reads `index.md` first),
documented in `defaults/CLAUDE.md`. The shipped skill lives in `defaults/skills/query/`.

## Requirements
### Requirement: `/query` answers a user request from the wiki

Claude Code SHALL provide a `/query` skill that takes a natural-language request and
answers it using the wiki. The skill SHALL read `index.md` first, select the relevant
pages from the index, read those pages, and answer from their contents. It SHALL NOT
answer from memory when the wiki contains the relevant pages.

#### Scenario: A question is answered from wiki pages

- **WHEN** the user runs `/query` with a question the wiki has pages about
- **THEN** Claude reads `index.md`, opens the relevant pages, and answers from their contents

#### Scenario: Index is consulted before individual pages

- **WHEN** `/query` runs
- **THEN** Claude reads `index.md` before opening any individual page

### Requirement: `/query` cites the pages it used

The answer SHALL cite the wiki pages it drew from, by slug, so the user can verify and
navigate to the source pages.

#### Scenario: Answer lists its sources

- **WHEN** `/query` produces an answer from two pages
- **THEN** the answer names both page slugs it used

### Requirement: `/query` gap is reported plainly

WHEN the wiki has no page relevant to the request, Claude SHALL say so in plain language
rather than fabricating an answer, and MAY suggest ingesting a relevant source.

#### Scenario: No relevant page exists

- **WHEN** `/query` is asked about a topic with no matching page in the index
- **THEN** Claude states the wiki has nothing on it and suggests ingesting a source

### Requirement: `/query` uses generic file tools only

The skill SHALL use only generic list/read/search file operations. It SHALL NOT build or
require a search index, and SHALL NOT use vector embeddings at any point.

#### Scenario: No search index or embeddings are used

- **WHEN** `/query` runs
- **THEN** it navigates via `index.md` and file reads only, with no embeddings and no separate search index

