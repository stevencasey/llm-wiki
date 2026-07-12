# Tiering Decision: Starting Point vs. Non-Technical Enablement

## Why

Comparing this project's specs against Karpathy's own documented LLM-wiki pattern (see
`deep-analysis.md`) showed the core schema — page types, wikilinks, index,
`CLAUDE.md`-as-contract — is a faithful match. That part earned its place. But the specs
had also inherited two things Karpathy's own use case never needed:

1. His most advanced feature (the evaluation loop: golden dataset + LLM-as-Judge) was
   built after years of his own usage revealed the need for it. This project had
   speced it as a phase-2 requirement from day one, before running the system once.
2. Packaging this as an installable product for non-technical strangers is a goal added
   on top of his pattern, not something his pattern itself required — he built it for
   himself, a technical user operating his own tools.

## Decision

Split the specs into two tiers:

- **Tier 1 (starting point)** — replicate Karpathy's actual pattern: wiki schema +
  `CLAUDE.md` + a folder + generic filesystem access via Claude Desktop/Claude Code. No
  custom server, no CLI, no backend. Usable today by a technical person.
- **Tier 2 (non-technical enablement)** — the delta needed to make Tier 1 usable by
  someone who can't configure MCP by hand. Deliberately unscoped on mechanism until
  Tier 1 has been used for real and its actual gaps (not guessed ones) are known.

## What this changed

- Dropped the custom `wiki` CLI and the bespoke MCP server (`ls`/`read`/`grep`/`search`)
  in favour of the official filesystem MCP server for Tier 1.
- Removed the tech-stack lock-in (Python/uv, mandatory Claude-API-call parameters) from
  the constitution — Tier 2's implementation isn't decided yet, and Tier 1 doesn't call
  the API at all (it's Claude, in chat, using file tools).
- Flattened the wiki schema's lint checklist from a blocker/warning taxonomy to one
  list, and made the `provenance:` field encouraged instead of required — both were
  process designed before a single page existed.
- The evaluation loop remains explicitly out of scope until Tier 1 has real usage data
  to build a golden dataset from — see `deep-analysis.md` for why that's hard to
  bootstrap otherwise.
