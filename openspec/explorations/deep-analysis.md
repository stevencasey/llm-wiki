# Deep Analysis: User's LLM OS Architecture

## Context

The user shared a Karpathy-style "LLM OS" architecture (3 subsystems: ingestion pipeline with quality gate, MCP file system query interface, continuous evaluation loop with golden dataset and LLM-as-Judge). This document deepens the analysis of trade-offs, failure modes, and gaps across all three subsystems.

---

## Subsystem 1: Ingestion Pipeline

### What the architecture gets right
The quality gate is the correct design. Without it, a Karpathy wiki degrades silently — broken cross-links and inconsistent naming accumulate and the LLM increasingly fails to navigate correctly. Blocking at ingest is far cheaper than diagnosing failures after the fact.

### The schema.md / CLAUDE.md is the most important artifact
Karpathy's own implementation names this the "CLAUDE.md" (or AGENTS.md). It is the operational spec for every agent behaviour:
- Directory structure and naming conventions
- What constitutes an entity page vs concept page vs synthesis page
- Required YAML frontmatter fields (source, date, tags, provenance)
- Cross-link rules (when to create a `[[slug]]` link, what slugs are valid)
- The `lint` operation: orphaned pages, broken cross-references, missing source provenance

The ingestion evaluator in the diagram IS the lint step — but the schema that defines what "valid" means is the real lever. If the schema is vague, the evaluator has nothing to check against.

### What the diagram doesn't show: ~10–15 wiki pages touched per source ingest
When a new source lands, the LLM doesn't just write one summary page. It updates the index, creates or updates entity pages, updates concept pages, and flags contradictions. This is expensive per source but is what makes the wiki compound rather than grow linearly.

---

## Subsystem 2: Query Interface (MCP File System Server)

### Scale limit: the no-vector bet works to ~few hundred pages
Research confirms that index-first navigation (read the index, load relevant pages) scales to a few hundred pages without hitting context limits. Beyond that, two problems emerge:

1. **Lost in the Middle**: Even with 1M token context windows, 11/13 major models drop below 50% baseline accuracy at 32K tokens when they can't use surface-level pattern matching (NoLiMa benchmark, ICML 2025). Loading hundreds of files into context degrades synthesis quality.

2. **Navigation cost**: The LLM must issue multiple tool calls (ls → read index → grep → read file). At scale, this chain gets longer and more expensive per query.

### The index file is the unmentioned load-bearing component
The diagram shows `ls, cat, grep, read` as MCP primitives. But at any non-trivial scale, the LLM needs an index to decide which files to open. A well-maintained index (updated on every ingest) is the substitute for vector search in this architecture. Without it, the LLM either loads everything (context overflow) or greps blindly (slow, misses semantic neighbours).

### The no-vector bet is correct but conditional
Pure grep + structural clarity works IF:
- The index is maintained and accurate
- File naming is semantically rich (not `doc-2024-03-15.md` but `sentinel-sca-policy-enforcement.md`)
- Tags in YAML frontmatter are consistent and curated

If any of these slip, vector search would have compensated. This is why the structural feedback loop in the evaluation section is critical — it's what keeps the taxonomy honest.

### Optional: a search primitive without abandoning the philosophy
A BM25 index (not vector embeddings) could be added as a `search(query)` MCP tool without violating the no-vector philosophy. It's deterministic, explainable, and fast. The distinction matters: embeddings encode semantic similarity into opaque vectors; BM25 is just term frequency over your existing markdown. This would extend the scale ceiling significantly.

---

## Subsystem 3: Continuous Evaluation Loop

### The golden dataset is the hardest part to build
A golden dataset of production queries paired with ground-truth answers requires:
- Knowing what "correct" looks like before the system is built
- Avoiding circular reasoning (the wiki generates the answers; the answers validate the wiki)
- Human curation — at least initially

Practical starting point: use your existing research docs as ground truth. E.g., "What are the key features of Sentinel SCA?" with the answer derived from `audit_research/sentinel_sca_overview.md`. This bootstraps the dataset from known-good content.

### LLM-as-Judge reliability issues (June 2026 research)
The judge has systematic failure modes that will produce false signals in your feedback loops:

| Bias | Effect | Mitigation |
|---|---|---|
| Position bias | Favours answer in slot A in pairwise comparisons | Alternate order across runs |
| Verbosity bias | Longer answers score higher regardless of quality | Add explicit length-normalised scoring |
| Non-determinism | Same prompt, different scores across runs | Average 3+ runs; pin temperature |
| Domain calibration | Weak on specialised content | Use domain-specific rubrics in judge prompt |

A 2026 study found frontier models exceed 50% error rates on challenging bias benchmarks. **Implication for the architecture**: the evaluation loop will generate noisy signals. Don't trigger structural changes from a single eval run — require consistent failure across multiple runs before acting.

### The two feedback loops have asymmetric costs
- **Accuracy loop** (tune system prompt / schema) — cheap, immediate, no re-ingestion needed
- **Structural loop** (rename files, retag, reorganise ontology) — expensive, requires re-ingestion or manual reorganisation of existing wiki pages

The architecture correctly separates these. The practical rule: always try the accuracy loop first. Only escalate to structural changes when the model consistently opens the right files but still fails to synthesise.

### Missing: contradiction detection
The schema lint in Karpathy's reference implementation includes a `--deep` flag that cross-checks tag-overlapping pages for factual contradictions. This is valuable but absent from the diagram. As the wiki grows, contradictions between early and late pages become a silent accuracy problem that neither the ingestion evaluator nor the standard eval loop catches.

---

## Summary: Gaps and Risks

| Gap | Risk | Recommended fix |
|---|---|---|
| Schema.md not shown explicitly | Evaluator has nothing concrete to check against | Define schema first, before building the evaluator |
| Index file not in the diagram | Scale breaks without it | Add an auto-updated `index.md` to the wiki as a required artifact |
| No BM25/search primitive | Hard scale ceiling at ~200–300 pages | Add `search()` MCP tool (BM25, not vectors) |
| LLM-as-Judge is noisy | False structural changes triggered by bias, not real failures | Require N consecutive failures before triggering structural loop |
| Contradiction detection absent | Silent accuracy degradation as wiki grows | Add lint `--deep` step on a schedule |
| Golden dataset bootstrap is circular | Eval loop starts with no signal | Seed from existing known-good research docs |

---

## Where this sits vs my original suggestions

My three options (Obsidian + plugin, Claude Code custom pipeline, qmd) covered **storage and retrieval only**. The user's architecture is two layers higher:

- Layer 1 (storage + retrieval) — covered by all three options
- Layer 2 (ingestion quality gate) — only partially in the schema lint of Karpathy's gist; absent from my recommendations
- Layer 3 (evaluation loop) — absent from all my recommendations; this is the system-level differentiator

The Obsidian plugin is a good way to explore the pattern and develop the schema.md intuitively before committing to building the evaluation infrastructure. That's still the right starting point.

---

Sources:
- [NoLiMa benchmark / context window performance](https://atlan.com/know/llm-context-window-limitations/)
- [LLM-as-Judge failure modes 2026](https://futureagi.com/blog/llm-as-judge/)
- [LLM-as-Judge reliability without validity study](https://arxiv.org/html/2606.19544)
- [Karpathy LLM Wiki production pattern](https://aaronfulkerson.com/2026/04/12/karpathys-pattern-for-an-llm-wiki-in-production/)
- [Beyond RAG: Karpathy's compounding knowledge](https://levelup.gitconnected.com/beyond-rag-how-andrej-karpathys-llm-wiki-pattern-builds-knowledge-that-actually-compounds-31a08528665e)
- [Enterprise reality check on LLM Wiki](https://www.innobu.com/en/articles/karpathy-llm-wiki-second-brain-enterprise-reality.html)
