# MCP File System Server Spec

The MCP server exposes the user's wiki directory to Claude Desktop as a set of
file system primitives. It is the query interface for subsystem 2.

---

## User-facing behaviour

When connected via Claude Desktop, the user can ask Claude questions about their wiki
and Claude will use the MCP tools to navigate and retrieve the relevant pages.

The server exposes these tools:

**`wiki_ls <directory>`** — Lists pages in a wiki directory (entities/, concepts/, etc.)
with their titles from frontmatter. Calling with no argument lists the root.

**`wiki_read <slug>`** — Returns the full content of a page by slug (filename without
`.md`). Resolves the correct directory from the page's type or by searching all directories.

**`wiki_grep <pattern>`** — Full-text search across all wiki pages. Returns matching
lines with their file paths and surrounding context.

**`wiki_search <query>`** — BM25 keyword search across page titles, tags, and body text.
Returns ranked results with one-line descriptions. This is the preferred tool for
open-ended queries; `wiki_grep` is for exact-match lookups.

**`wiki_index`** — Returns the full content of `index.md`. This is the first tool Claude
should call on any query — it provides a map of the wiki before opening individual pages.

---

## Constraints

- All tools are read-only. No MCP tool may write to the wiki — writes happen through the
  ingest pipeline only.
- `wiki_search` must use BM25, not vector embeddings. The implementation must be
  deterministic and produce the same ranking for the same query and corpus.
- Tools must return plain text (markdown). No JSON wrappers in the tool response body.
- If a slug does not resolve to an existing page, `wiki_read` returns a clear error
  message, not an empty response or a stack trace.
- The server must start in under 2 seconds on a standard laptop with a 500-page wiki.
- Phase 2 addition: `wiki_suggest` — proposes schema improvements based on eval loop
  output. Not in scope for phase 1.
