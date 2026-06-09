---
name: slims-v9-db-structure
description: SLiMS 9 Bulian database structure guide for reading, documenting, tracing, and explaining the SQL schema. Use this skill to understand SLiMS v9 table purposes, follow relationships between bibliography, item, member, circulation, master, OPAC search, and admin tables, map SQL fields to library workflows, or explain schema details from the raw SQL.
---

# SLiMS v9 DB Structure

## Quick Start

- Confirm the target is SLiMS 9 Bulian.
- Read [references/schema-overview.md](references/schema-overview.md) before writing SQL, documentation, or integration notes.
- Switch to `dspace-v6-db-structure` if the task is about DSpace, not SLiMS.

## Working Rules

- Identify the business area first: catalog, copies, loans, members, settings, search, or admin.
- Find the core table before expanding to bridge and lookup tables (e.g., `biblio_author`, `biblio_topic`, `mst_*`).
- Treat `senayan.sql` as the authoritative baseline.
- Treat table relationships as logical (application-enforced), not database-enforced FK constraints — many tables use `MyISAM`.
- Separate bibliographic records (`biblio` = title/work) from physical copies (`item` = exemplar).
- Decode coded fields via master tables (`mst_*`) instead of guessing their meaning.
- Prefer structural explanation over speculative business logic. If a column purpose is unclear, say so.
- Check denormalized tables (`search_biblio`, `loan_history`) only after understanding the core transaction tables.
- Some fields store serialized values or JSON — note this before proposing direct SQL changes.

## Bundled Resource

- `references/schema-overview.md`: Table families, key fields, relationship map, and storage caveats from the official `senayan.sql`.
