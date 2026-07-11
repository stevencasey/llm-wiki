# Wiki Schema Spec

The wiki schema defines the structure, naming conventions, page types, and quality rules
that govern the user's personal knowledge base. It is the source of truth for all agent
behaviour when reading or writing wiki pages.

The canonical operational spec for Claude is `wiki/CLAUDE.md` inside the user's wiki
directory. The canonical human reference is `wiki/schema.md`. This spec describes the
contract those files must fulfil — not the files themselves.

---

## User-facing behaviour

**Page types.** The wiki contains five page types, each with a dedicated directory:

| Type | Directory | When it exists |
|---|---|---|
| Entity | `entities/` | A named, discrete thing the user refers to repeatedly (person, company, tool, place) |
| Concept | `concepts/` | An abstract idea, framework, methodology, or recurring term |
| Source | `sources/` | An external input the user has ingested (article, book, paper, podcast) |
| Synthesis | `synthesis/` | The user's own analysis drawing on two or more sources |
| Note | `notes/` | Personal capture: a thought, meeting notes, or a daily log |

**Filenames.** Every page has a semantically meaningful filename in kebab-case. Date-based
filenames are only permitted for daily notes (`notes/YYYY-MM-DD.md`).

**Frontmatter.** Every page opens with a YAML frontmatter block containing at minimum:
`type`, `title`, `tags`, `created`, `updated`. Source pages additionally require `author`,
`date_published`, and `url` or `isbn`. Note pages additionally require `promoted`.

**Wikilinks.** Cross-references use `[[slug]]` syntax where slug is the filename without
`.md`. A wikilink to a non-existent page is never permitted; a stub must be created first.

**Index.** `index.md` is a single file at the wiki root listing all entities, concepts,
sources, synthesis pages, and recent notes with one-line descriptions. It is updated on
every ingest and every new daily note. It is the primary navigation tool for the LLM.

---

## Lint contract

The quality gate runs after every ingest. Results are classified as blockers or warnings.

**Blockers** (ingest does not complete until resolved — Claude resolves inline):
- Broken wikilinks: any `[[slug]]` where `slug.md` does not exist
- Missing frontmatter: any page missing a required field for its type
- Orphaned pages: any entity, concept, or synthesis page with zero inbound wikilinks

**Warnings** (reported, do not block):
- Missing provenance: entity or concept pages with no `provenance:` field and no inbound
  link from a source page

**Exemptions:**
- Note pages are exempt from the orphan check
- Note pages are exempt from the provenance warning

---

## Promotion contract

A note page is considered promotable when the user requests promotion or when the same
idea has appeared independently in two or more notes. Promotion creates a permanent page
in the appropriate directory and links back to the originating note. Once all key content
in a note has been promoted, `promoted: true` is set in its frontmatter and it is removed
from the `## Notes` section of `index.md`.

---

## Constraints

- No vector embeddings at any layer of the schema or lint implementation.
- The schema is defined in `wiki/CLAUDE.md` (agent-facing) and `wiki/schema.md`
  (human-facing). Both files travel with the user's wiki, not with the project code.
- Default copies of both files live in `defaults/` in the project repo and are copied
  into the user's wiki on `wiki init`. The user may edit their copies freely.
- The project must never overwrite a user's edited `CLAUDE.md` or `schema.md` on upgrade.
- The five content directories are fixed. No other directories may be created inside the
  wiki root by the system.
