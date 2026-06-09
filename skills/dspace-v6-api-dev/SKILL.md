---
name: dspace-v6-api-dev
description: DSpace 6.x REST API guide for building, debugging, documenting, or reviewing integrations against the legacy `/rest` API. Use this skill whenever the user mentions DSpace 6 REST, `/rest`, `JSESSIONID`, login/logout, status, `expand`, pagination, or endpoint families such as communities, collections, items, bitstreams, handle, hierarchy, registries, or reports. Do not use it for DSpace 7+ `/server/api` or database-structure questions.
---

# DSpace v6 API Dev

## Quick Start

- Confirm the target is DSpace 6.x with base path `/rest`.
- Read [references/rest-api.md](references/rest-api.md) before writing code, examples, or docs.
- Switch to `dspace-v6-db-structure` if the task is about tables, joins, or schema tracing.

## Working Rules

- Default to `Accept: application/json` unless XML is explicitly needed.
- Treat auth as cookie-based: `POST /login` returns `JSESSIONID`; reuse that cookie; invalidate with `POST /logout`.
- Check the endpoint family first, then the exact object shape from the reference.
- Use `limit` and `offset` on paginated endpoints; do not expect a total count.
- Use targeted `?expand=` values in real integrations; reserve `?expand=all` for inspection.
- Treat any write or restricted read as requiring a valid authenticated session.
- Prefer a minimal `curl` reproduction when behavior is unclear.

## High-Signal Notes

- `POST /items/{item id}/bitstreams?name=...` sends raw file bytes in the request body.
- `PUT /items/{item id}/metadata` replaces prior matching metadata entries.
- `PUT /bitstreams/{bitstream id}` updates metadata; `PUT /bitstreams/{bitstream id}/data` updates file content.
- Shibboleth path naming in legacy docs is inconsistent; verify deployed config before implementing it.
- DSpace 6 documentation is legacy; do not reuse this skill for DSpace 7+.

## Bundled Resource

- `references/rest-api.md`: Endpoint map, session workflow, pagination rules, object skeletons, and source-derived caveats from `REST_API_DSpace_6.x_Documentation_LYRASIS_Wiki.pdf`.
