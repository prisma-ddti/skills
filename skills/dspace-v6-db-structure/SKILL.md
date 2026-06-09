---
name: dspace-v6-db-structure
description: DSpace 6.x database structure reference for reading, documenting, tracing, and explaining the relational schema shown in the supplied schema diagram. Use this skill to identify DSpace 6 tables, explain how communities, collections, items, bundles, bitstreams, metadata, identifiers, workflow, and permissions relate to each other, trace storage around `dspaceobject`, or answer table-level questions that are separate from REST API behavior.
---

# DSpace v6 DB Structure

## Quick Start

- Confirm the target is DSpace 6 relational schema, not DSpace 7+.
- Read [references/rdbms-schema.md](references/rdbms-schema.md) before explaining table purpose, joins, or object storage.
- Use `dspace-v6-api-dev` instead when the task is about `/rest` API behavior.
- Treat the schema diagram (`assets/rdbms-dspace6.png`) as the primary source for table grouping and relationships.

## Working Rules

- Identify the schema region first: hierarchy, item storage, bundles/bitstreams, metadata, identifiers, permissions, or eperson/group workflow.
- Start from `dspaceobject` when tracing how multiple object tables connect through UUID-backed identity.
- Expand through join tables (`community2community`, `community2collection`, `collection2item`, `item2bundle`, `bundle2bitstream`) before inferring business meaning.
- Separate storage families from operational overlays (metadata, permissions, identifiers, harvest, workflow, checksums).
- Keep REST API semantics separate from storage semantics.
- Explain relationships in terms of tables and join paths, not guessed ORM behavior.
- Prefer exact table names from the diagram. If column-level semantics are not visible, say so.
- Use PostgreSQL migration directory only when the diagram needs schema-history explanation.

## Bundled Resources

- `references/rdbms-schema.md`: Schema backbone, object-to-table mapping, join paths, and migration landmarks.
- `assets/rdbms-dspace6.png`: DSpace 6 schema diagram.
