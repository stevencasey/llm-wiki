# llm-wiki

An open-source personal knowledge base where Claude is the primary reader and writer.
You paste a URL or describe a thought in Claude Desktop chat; Claude ingests it, updates
the relevant pages, keeps an index, and enforces a lightweight quality gate — no vector
database, no manual filing.

This project is built spec-first with [OpenSpec](https://openspec.dev). **There is no
installable system yet** — the repo currently contains the wiki schema and the OpenSpec
specs that will drive implementation. This README will grow a "Running the wiki" section
once `src/` exists.

---

## Project status

Phase 1 (in progress): the wiki schema, the ingest pipeline, and the MCP file system
server. Phase 2 (not started): the continuous evaluation loop and the self-improvement
agent that proposes schema changes. See `openspec/config.yaml` for the full scope split.

---

## What's installed, what you need

| Tool | Why | Check you have it |
|---|---|---|
| [Claude Code](https://claude.com/claude-code) | Drives the `/opsx:*` spec workflow and will implement the system | `claude --version` |
| [OpenSpec CLI](https://openspec.dev) | Validates specs, manages change proposals | `openspec --version` |
| Git | You're reading this in a git repo | `git --version` |

Install the OpenSpec CLI if you don't have it:

```bash
npm install -g @fission-ai/openspec@latest
```

Nothing else is required yet — once the Python implementation starts, this section will
list the runtime dependencies (Python version, `uv`, etc.) alongside `pyproject.toml`.

---

## Repo structure

```
llm-wiki/
├── defaults/
│   ├── CLAUDE.md            ← wiki operational spec (copied into a user's wiki on init)
│   └── schema.md             ← wiki schema, human-facing reference with rationale
├── openspec/
│   ├── config.yaml           ← project constitution: tech stack, product language,
│   │                           architectural constraints, per-artifact writing rules
│   ├── specs/                ← current, agreed specs — one folder per domain
│   │   ├── wiki/spec.md      ← wiki schema contract (page types, lint, promotion)
│   │   ├── cli/spec.md       ← `wiki` command-line interface
│   │   ├── ingest/spec.md    ← ingest pipeline (5-step procedure, capture flow)
│   │   └── mcp/spec.md       ← MCP file system server (ls, read, grep, search, index)
│   ├── changes/              ← in-flight change proposals (empty until one is opened)
│   └── explorations/
│       └── deep-analysis.md  ← original research memo: gap analysis, benchmark
│                                citations, rationale behind the no-vector bet
└── .claude/                  ← Claude Code slash commands and skills for the
                                 `/opsx:*` workflow (explore, propose, apply, archive)
```

`defaults/CLAUDE.md` and `defaults/schema.md` are **not** this project's own dev
instructions — they're the spec for the *wiki itself*, templated to be copied into a
user's wiki directory. Don't confuse them with a project-root `CLAUDE.md`, which doesn't
exist yet and will hold actual build/test instructions once there's code to describe.

---

## How to read and review the specs

Start with `openspec/config.yaml` — it's the constitution. It defines the tech stack,
the product vocabulary every spec must use (`ingest`, `capture`, `promote`, `lint` —
not synonyms), the architectural constraints that are non-negotiable (no vector
embeddings, index-first navigation), and the writing rules specs/design docs/tasks must
follow. Read this first; everything else assumes it.

Then read `openspec/specs/wiki/spec.md` — the wiki schema is the foundation the other
three specs depend on. `cli/spec.md`, `ingest/spec.md`, and `mcp/spec.md` can be read in
any order after that; each is self-contained with a "User-facing behaviour" section and
a "Constraints" section.

`openspec/explorations/deep-analysis.md` is background, not a spec — it's the original
research and trade-off analysis that the specs were distilled from. Read it if you want
the *why* behind a decision (e.g. why the provenance lint check is a warning and not a
blocker, why LLM-as-Judge needs multi-run averaging). It's not kept in sync with the
specs going forward, so treat it as historical context, not current truth.

To validate a spec's structure rather than just read it:

```bash
openspec validate <domain>   # e.g. openspec validate wiki
```

---

## Proposing a change

This repo uses OpenSpec's delta-spec workflow — changes are proposed, reviewed, and
applied through Claude Code, not written directly into `openspec/specs/`.

```
/opsx:explore    optional — think through a fuzzy idea before anything is drafted
/opsx:propose    draft a change: proposal, delta specs, design, tasks — for your review
/opsx:apply      implement the approved change
/opsx:archive    fold the delta into openspec/specs/ and file the change away
```

Each in-flight change gets its own folder under `openspec/changes/`. Nothing in
`openspec/specs/` should be hand-edited directly — it's the merge target, not the
draft surface.

---

## Contributing

Phase 1 scope only for now (see `openspec/config.yaml`): the wiki schema, the ingest
pipeline, and the MCP server. Don't open change proposals for the evaluation loop or the
self-improvement agent (phase 2) until phase 1 is implemented and phase 2's design is
separately approved.
