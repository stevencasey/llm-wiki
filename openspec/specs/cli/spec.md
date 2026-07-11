# CLI Spec

The `wiki` CLI is the primary interface for non-technical users to initialise and operate
their wiki outside of Claude Desktop chat.

---

## User-facing behaviour

**`wiki init <path>`** — Creates a new wiki at the given path. Copies `CLAUDE.md` and
`schema.md` from `defaults/`, creates the five content directories, and creates an empty
`index.md`. Reports the path and a one-line next step ("Run `wiki serve` to connect to
Claude Desktop, then start adding pages by pasting a URL in chat").

**`wiki ingest <url-or-file>`** — Ingests a single external source. Runs the 5-step
ingest procedure and reports which pages were created or updated, and whether lint passed.
Non-technical users are not expected to use this directly — it is called internally when
they ask Claude to add a source via chat.

**`wiki serve`** — Starts the MCP file system server and prints the config snippet the
user needs to paste into Claude Desktop. Runs until interrupted.

**`wiki lint`** — Runs the standard lint checklist against the wiki and reports all
blockers and warnings. Does not fix anything — this is a diagnostic command for the user
to see the current state.

**`wiki watch`** (optional) — Watches `notes/inbox/` for new files and auto-ingests them.

---

## Constraints

- All commands require a wiki path. If not passed as an argument, the CLI reads it from
  `~/.llm-wiki/config.yaml`. If neither is set, the CLI exits with a plain-English error
  and a suggested fix.
- Error messages must not expose stack traces to non-technical users. Log details to a
  file; show a one-line human message with a next action.
- `wiki init` must refuse to overwrite an existing wiki unless `--force` is passed.
- `wiki init` must never overwrite an existing `CLAUDE.md` or `schema.md` even with
  `--force` — those are the user's files.
