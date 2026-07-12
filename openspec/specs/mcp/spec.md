# Wiki Access Spec (Tier 1)

Tier 1 needs no custom server. Claude accesses the wiki directory the same way it
accesses any local folder: through the official filesystem MCP server
(`@modelcontextprotocol/server-filesystem`) pointed at the wiki root, or direct local
file access when running in Claude Code. This spec records what's needed *beyond*
generic file access — not a reimplementation of ls/read/grep, which already exist.

---

## User-facing behaviour

- Claude reads `index.md` before opening individual pages on any wiki query — this is a
  `CLAUDE.md` instruction (see `defaults/CLAUDE.md`), not a tool-level restriction.
- No additional tooling is required for Tier 1. Generic filesystem list/read/search
  operations are sufficient at the scale a personal wiki reaches early on (a few hundred
  pages — see `openspec/explorations/deep-analysis.md` for the research behind that
  ceiling).

---

## Deferred: a search primitive

If real usage shows the index plus generic file tools no longer surface the right
pages, add a `search(query)` capability using BM25 — not vector embeddings. This is
explicitly deferred, not a Tier 1 requirement. Don't build it speculatively.

---

## Constraints

- No vector embeddings, ever, at any tier.
- No custom MCP server for Tier 1. If a future spec proposes one, it must justify what
  generic file access and the official filesystem server can't already do.
