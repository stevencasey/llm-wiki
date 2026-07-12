## ADDED Requirements

### Requirement: On-demand ingest sweep of a drop location

When invoked inside a Claude Code session, Claude SHALL scan the designated drop
location and process every file that is new or has changed since it was last processed,
and SHALL skip every file that is unchanged. The sweep is triggered by invocation only;
there SHALL be no background process, watcher, or daemon between sessions (Tier 1).

#### Scenario: A newly added file is ingested

- **WHEN** a file exists in the drop location whose path is not recorded in the ledger
- **AND** the user invokes the sweep
- **THEN** Claude ingests that file following the ingest procedure and records it in the ledger

#### Scenario: An unchanged file is skipped

- **WHEN** a file's recorded content hash matches its current content
- **AND** the user invokes the sweep
- **THEN** Claude does not re-ingest it and reports it as already processed

#### Scenario: The sweep only runs when invoked

- **WHEN** no sweep has been invoked
- **THEN** no file is ingested and no background process is running

### Requirement: The user's own files are never modified or removed

The sweep SHALL NOT edit, rename, move, or delete any file in the drop location. Marking
a file as processed SHALL be done outside the file.

#### Scenario: Source file is byte-for-byte unchanged after a sweep

- **WHEN** the sweep ingests a file
- **THEN** the original file's contents and location are identical to before the sweep

### Requirement: Processed state is tracked in an external ledger

Claude SHALL record processed state in a ledger stored separately from the user's files.
For each processed file the ledger SHALL record its path, a content hash, and the wiki
pages ingest produced from it. The ledger SHALL be the sole record of what has been
processed.

#### Scenario: Ledger records identity and derived pages

- **WHEN** a file is ingested
- **THEN** the ledger contains that file's path, content hash, and the slugs of the pages created or updated for it

#### Scenario: A second sweep uses the ledger to skip work

- **WHEN** the sweep runs again with no files changed
- **THEN** Claude reads the ledger, finds every hash unchanged, and ingests nothing

### Requirement: Changed files are re-ingested and updated in place

WHEN a file's current content hash differs from the hash recorded in the ledger, Claude
SHALL re-ingest the file and update the pages previously derived from it, and SHALL NOT
create duplicate pages for the same file.

#### Scenario: An edited file updates its existing pages

- **WHEN** a previously-ingested file is changed and the sweep runs
- **THEN** Claude updates the pages recorded in the ledger for that file to reflect the new content
- **AND** does not create a second source page for the same file

#### Scenario: A moved but unchanged file is not duplicated

- **WHEN** a file is relocated within the drop location but its content hash is unchanged
- **THEN** Claude recognizes it by content hash and does not create duplicate pages

### Requirement: Each dropped file is classified as source or note

The sweep SHALL classify each file before processing it. An external document SHALL be
ingested (source page plus entity, concept, index, and lint steps). A personal note SHALL
be captured following the notes rules, not fully ingested. The sweep SHALL NOT invent new
directories or page types.

#### Scenario: An external document becomes a source page

- **WHEN** a dropped file is an external article, paper, or document
- **THEN** Claude ingests it into `sources/` and updates entities, concepts, and the index

#### Scenario: A personal note is captured, not ingested

- **WHEN** a dropped file is a personal note
- **THEN** Claude captures it per the notes rules and does not run the full source ingest steps

### Requirement: Sweep ingest is idempotent against a stable source identity

Ingest invoked by the sweep SHALL be keyed to a stable identity for each source (its
content hash, recorded in the ledger). Re-ingesting the same source SHALL update the pages
that source already produced rather than creating new ones, making "ingesting the same
source twice must not create duplicate pages" a testable requirement.

#### Scenario: Re-ingesting an unchanged source creates no pages

- **WHEN** the sweep encounters a source whose hash matches its ledger entry
- **THEN** no new pages are created and existing pages are unchanged

#### Scenario: Re-ingesting a changed source updates in place

- **WHEN** the sweep re-ingests a source with changed content
- **THEN** the pages recorded for it are updated in place and no duplicate pages are created

### Requirement: Batch ingest does not stop to ask

When ingest runs as one file inside a sweep, it SHALL fix every lint problem inline (as in
interactive ingest) and continue to the next file. It SHALL NOT pause to ask the user to
resolve a lint failure mid-sweep.

#### Scenario: A sweep with lint problems completes without prompting

- **WHEN** ingesting a swept file surfaces lint problems
- **THEN** Claude fixes them inline and continues to the next file without asking the user

### Requirement: The sweep stays within Tier 1 constraints

The sweep SHALL use only generic filesystem access and the ingest procedure. It SHALL NOT
introduce a backend, a search index, or vector embeddings, and SHALL NOT process file
types beyond what Claude can already read in a session.

#### Scenario: No new infrastructure is required

- **WHEN** the sweep runs
- **THEN** it uses only file read/list operations and the existing ingest procedure, with no server, index, or embeddings
