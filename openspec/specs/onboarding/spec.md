# Onboarding Spec (Tier 2)

This is Tier 2: the requirements for letting a non-technical user get a working wiki
connected to Claude without touching a terminal or hand-editing configuration. It is
**not scoped yet on mechanism** — Tier 1 (see `wiki/spec.md`, `ingest/spec.md`,
`mcp/spec.md`) needs to be built and used for real first, so this spec is written
against actual gaps rather than guessed ones.

---

## User-facing behaviour (outcomes Tier 2 must satisfy — mechanism not yet decided)

- A non-technical user can get a wiki folder set up with the correct structure and the
  default `CLAUDE.md`/`schema.md` without running any command-line tool.
- A non-technical user can connect that folder to Claude Desktop without hand-editing
  JSON configuration.
- If something goes wrong, the user sees a plain-English explanation and a next step —
  never a stack trace or a raw error code.
- A non-technical user never needs to know what MCP, YAML, or frontmatter are to use
  the wiki day to day.

---

## Explicitly not decided here

- Whether this is a CLI, a native app, a Claude Desktop extension, a web installer, or
  something else.
- What language or runtime it's built in.
- Whether a drop-folder ("inbox") capture path is needed. Chat-based capture — pasting
  a URL or describing a thought directly to Claude — is the validated Tier 1 mechanism;
  assume it's sufficient until Tier 1 use proves otherwise.

---

## Constraints

- Nothing in this spec may be implemented before Tier 1 has been used for real, by an
  actual person, on actual sources.
- Whatever setup mechanism is chosen, it must never overwrite a user's edited
  `CLAUDE.md` or `schema.md`.
