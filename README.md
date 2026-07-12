# llm-wiki

## Quick start: reviewing the specs

The specs are plain markdown — no install needed to read them. Open in this order:

1. [`openspec/config.yaml`](openspec/config.yaml) — the constitution: the two-tier model, terminology, non-negotiable constraints
2. [`openspec/specs/wiki/spec.md`](openspec/specs/wiki/spec.md) — the wiki schema (everything else depends on this)
3. [`openspec/specs/ingest/spec.md`](openspec/specs/ingest/spec.md), [`mcp/spec.md`](openspec/specs/mcp/spec.md) — Tier 1 (the starting point); [`onboarding/spec.md`](openspec/specs/onboarding/spec.md) — Tier 2 (non-technical enablement, intentionally unscoped for now)
4. [`openspec/explorations/`](openspec/explorations/) — background/rationale, not specs: `deep-analysis.md` (original research) and `tiering-rationale.md` (why Tier 1/Tier 2 split)

## To propose a change

Requires [Claude Code](https://claude.com/claude-code) and the [OpenSpec CLI](https://openspec.dev) (`npm install -g @fission-ai/openspec@latest`).

```
/opsx:explore    think through an idea (optional)
/opsx:propose    draft a change for review
/opsx:apply      implement an approved change
/opsx:archive    merge into openspec/specs/
```

Don't hand-edit `openspec/specs/` directly — it's the merge target for approved changes.

## Repo structure

```
defaults/CLAUDE.md, schema.md   the wiki's own spec (copied into a user's wiki during
                                 setup — not this project's dev instructions)
openspec/config.yaml            constitution
openspec/specs/                 current specs, one folder per domain
openspec/changes/               in-flight change proposals
openspec/explorations/          background research, not authoritative
.claude/                        Claude Code commands for the /opsx workflow
```
