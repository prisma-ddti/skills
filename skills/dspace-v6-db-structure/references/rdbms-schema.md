# DSpace 6.x RDBMS Schema Map

Migration source: `https://github.com/DSpace/DSpace/tree/dspace-6_x/dspace-api/src/main/resources/org/dspace/storage/rdbms/sqlmigration/postgres`

## Schema Backbone

| Concern                | Tables                                                                             | Notes                                                                |
| ---------------------- | ---------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| Repository identity    | `dspaceobject`                                                                     | Central UUID table connecting repository object families.            |
| Hierarchy joins        | `community2community`, `community2collection`, `collection2item`                   | Structural links for site, community, collection, and item placement |
| File packaging         | `item2bundle`, `bundle`, `bundle2bitstream`, `bitstream`                           | Storage path from repository item to file objects.                   |
| Metadata normalization | `metadataschemaregistry`, `metadatafieldregistry`, `metadatavalue`                 | Schema, field, and value layers.                                     |
| Access and identity    | `eperson`, `epersongroup`, `epersongroup2eperson`, `group2group`, `resourcepolicy` | Authentication, grouping, and policy enforcement.                    |
| External identifiers   | `handle`, `doi`                                                                    | Separate identifier tables pointing to repository resources.         |

## Object-to-Table Mapping

| Repository concept | Primary tables               | Supporting tables                                                                                                                                | Notes                                                          |
| ------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| Site               | `site`                       | `community2community`, `community2collection`                                                                                                    | Root repository structure.                                     |
| Community          | `community`, `dspaceobject`  | `community2community`, `community2collection`, `bitstream`                                                                                       | Community hierarchy and logo/file references.                  |
| Collection         | `collection`, `dspaceobject` | `community2collection`, `collection2item`, `subscription`, `harvested_collection`                                                                | Bridges hierarchy, workflow, and harvest configuration.        |
| Item               | `item`, `dspaceobject`       | `collection2item`, `item2bundle`, `workspaceitem`, `requestitem`, `workflowitem`, `versionitem`, `metadatavalue`                                 | Connects outward to workflow, files, and metadata.             |
| Bundle             | `bundle`                     | `item2bundle`, `bundle2bitstream`                                                                                                                | Packages one or more bitstreams under an item.                 |
| Bitstream          | `bitstream`                  | `bundle2bitstream`, `bitstreamformatregistry`, `fileextension`, `resourcepolicy`, `checksum_history`, `checksum_results`, `most_recent_checksum` | File objects, format info, policies, and checksum tracking.    |
| Metadata value     | `metadatavalue`              | `metadatafieldregistry`, `metadataschemaregistry`, `dspaceobject`                                                                                | Values attach to repository objects through registered fields. |
| User/group         | `eperson`, `epersongroup`    | `epersongroup2eperson`, `group2group`, `registrationdata`, `subscription`, `epersongroup2workspaceitem`                                          | Access, subscriptions, and workflow roles.                     |
| Policy             | `resourcepolicy`             | `eperson`, `epersongroup`, `dspaceobject`, `bitstream`                                                                                           | Governs visibility and actions for objects and files.          |
| Identifier         | `handle`, `doi`              | `dspaceobject`                                                                                                                                   | Resolves external names to internal resources.                 |

## Join Paths

1. **Hierarchy**: `community2community` and `community2collection` define tree structure; `collection2item` places items under collections.
2. **File**: `item` -> `item2bundle` -> `bundle` -> `bundle2bitstream` -> `bitstream`
3. **Metadata**: `dspaceobject` -> `metadatavalue` -> `metadatafieldregistry` -> `metadataschemaregistry`
4. **Access**: `resourcepolicy` -> `eperson` / `epersongroup` -> `group2group` / `epersongroup2eperson`
5. **Identifier**: `handle` / `doi` -> repository resource referenced in the identifier row

## Tracing Rules

1. Start with `dspaceobject` when the question spans multiple repository object types.
2. Use join tables before assuming direct foreign-key style links on the main tables.
3. Read metadata, identifiers, and policies as adjacent layers attached to core repository objects.
4. Treat workflow, request, and versioning tables as process-state overlays rather than replacements for `item`.
5. Use the diagram for relationship discovery; use other sources only when column-level behavior is needed.

## Migration Landmarks

Key migrations shaping the current DSpace 6 schema:

- `V6.0_2015.03.07__DS-2701_Hibernate_migration.sql` — Major restructure to Hibernate-managed schema.
- `V5.0_2014.09.26__DS-1582_Metadata_For_All_Objects.sql` — Metadata extended to all object types.
- `V6.0_2016.07.26__DS-3277_fix_handle_assignment.sql` — Handle assignment fix.

Full migration history at the source URL above. Use the diagram as current-state map; consult migrations only when a table/column needs historical explanation.
