---
name: query
description: Answer a question from this wiki. Reads index.md first, opens the relevant pages, answers from their contents, and cites the page slugs used. Use when the user runs /query or asks a question of the wiki.
allowed-tools: Read, Grep, Glob
license: MIT
---

Answer the user's question using this wiki. Follow the wiki's `CLAUDE.md` — never build a
search index and never use embeddings. Navigate index-first.

**Steps**

1. **Optionally sweep first.** If the user hasn't swept recently and `notes/` may contain
   unprocessed files, offer to run the notes sweep (see `CLAUDE.md` → "Sweeping the Notes
   Folder") so the answer reflects everything dropped in. Skip if they just want an answer.
2. **Read `index.md`.** Use it to choose the pages relevant to the question — do not open
   pages at random or read the whole wiki.
3. **Open the relevant pages** and read their contents. Follow `[[slug]]` wikilinks when a
   linked page is needed to answer.
4. **Answer from the pages.** Ground every claim in what the pages actually say.
5. **Cite the pages.** End with the list of page slugs you used, so the user can verify and
   navigate to them.
6. **If nothing relevant is in the index,** say so plainly and offer to ingest a source —
   do not fabricate an answer.

**Input**: the user's question follows `/query` (e.g. `/query what did I conclude about
retrieval-augmented generation?`).

**Guardrails**
- Index-first, always. Read `index.md` before any individual page.
- Generic file reads only — no search index, no embeddings.
- Cite the slugs you used.
