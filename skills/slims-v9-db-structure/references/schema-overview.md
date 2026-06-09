# SLiMS 9 Bulian Schema Overview

Source: `https://raw.githubusercontent.com/slims/slims9_bulian/refs/heads/master/install/senayan.sql`

## Storage Characteristics

| Concern             | Observation                                                                              |
| ------------------- | ---------------------------------------------------------------------------------------- |
| Database flavor     | MySQL / MariaDB                                                                          |
| Primary engine      | Many core tables use `MyISAM`                                                            |
| Charset/collation   | `utf8` with `utf8_unicode_ci`                                                            |
| Relationship style  | Most relationships are implied by naming and indexes, not hard foreign keys              |
| Search support      | `FULLTEXT` indexes and helper tables (`search_biblio`, `index_words`, `index_documents`) |
| Structured payloads | Some fields store serialized values, JSON, or text blobs rather than normalized rows     |

## Core Module Map

| Area                | Main tables                                                                                                                                            | Notes                                               |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| Bibliography        | `biblio`, `biblio_author`, `biblio_topic`, `biblio_attachment`, `biblio_relation`, `biblio_custom`, `biblio_log`                                       | Title-level records, subjects, authority links      |
| Copies / exemplars  | `item`, `item_custom`                                                                                                                                  | Physical copies tied to a bibliographic record      |
| Membership          | `member`, `member_custom`, `mst_member_type`                                                                                                           | Patron identity and membership policy               |
| Circulation         | `loan`, `loan_history`, `reserve`, `fines`, `mst_loan_rules`                                                                                           | Active loans, history, reservations, penalties      |
| Master/reference    | `mst_author`, `mst_topic`, `mst_gmd`, `mst_coll_type`, `mst_location`, `mst_item_status`, `mst_language`, `mst_place`, `mst_publisher`, `mst_supplier` | Lookup values used across modules                   |
| RDA/resource typing | `mst_content_type`, `mst_media_type`, `mst_carrier_type`, `mst_relation_term`, `mst_voc_ctrl`                                                          | Content/media/carrier and controlled vocabulary     |
| Serial control      | `serial`, `kardex`, `mst_frequency`                                                                                                                    | Subscriptions and issue tracking                    |
| OPAC/search         | `search_biblio`, `comment`, `files`, `files_read`, `index_words`, `index_documents`                                                                    | OPAC metadata, attachments, comments, indexing      |
| Admin/system        | `user`, `user_group`, `group_access`, `mst_module`, `setting`, `plugins`, `system_log`, `backup_log`, `user_tokens`                                    | Staff accounts, permissions, settings, logs         |
| Operations          | `stock_take`, `stock_take_item`, `visitor_count`, `holiday`, `content`                                                                                 | Stock opname, visitor logs, holidays, content pages |

## Key Table Roles

### `biblio`

Canonical bibliographic/title record. Stores title, edition, publication metadata, classification, notes, labels, RDA fields, and OPAC flags.

Key fields: `biblio_id`, `gmd_id`, `publisher_id`, `language_id`, `publish_place_id`, `content_type_id`, `media_type_id`, `carrier_type_id`, `opac_hide`, `promoted`. Fulltext indexes on title/note fields.

### `item`

Individual copy/holding. Parent: `item.biblio_id -> biblio.biblio_id`.

Key fields: `item_id`, `item_code` (unique, used by circulation), `inventory_code`, `call_number`, `coll_type_id`, `location_id`, `item_status_id`, `site`.

### `member`

Patron/member record. Uses string `member_id` (not auto-increment integer).

Key fields: `member_id`, `member_name`, `member_type_id`, `expire_date`, `is_pending`, `mpasswd`, `last_login`.

### `loan`

Active circulation transactions. Joins: `loan.item_code -> item.item_code`, `loan.member_id -> member.member_id`, `loan.loan_rules_id -> mst_loan_rules.loan_rules_id`.

Key fields: `loan_date`, `due_date`, `renewed`, `actual`, `is_lent`, `is_return`, `return_date`.

### `search_biblio`

Denormalized OPAC search helper. Contains flattened title, author, topic, publisher, location, items, and collection types. Not the authoritative source for catalog metadata.

## Common Relationship Patterns

| Pattern                | Tables                                                 | Notes                                                 |
| ---------------------- | ------------------------------------------------------ | ----------------------------------------------------- |
| Title to copy          | `biblio` -> `item`                                     | One bibliographic record can have many physical items |
| Title to authors       | `biblio` -> `biblio_author` -> `mst_author`            | Many-to-many authority linkage                        |
| Title to topics        | `biblio` -> `biblio_topic` -> `mst_topic`              | Many-to-many subject linkage                          |
| Title to files         | `biblio` -> `biblio_attachment` -> `files`             | Attachment mapping separate from file metadata        |
| Member to loan         | `member` -> `loan`                                     | Uses `member_id` string key                           |
| Item to loan           | `item` -> `loan`                                       | Uses `item_code`, not `item_id`                       |
| User to access rights  | `user` -> `user_group` / `group_access` / `mst_module` | Staff permission is group-module based                |
| Serial title to issues | `serial` -> `kardex`                                   | Issue/receipt tracking                                |

## Master Table Lookup

| Field                     | Lookup table       |
| ------------------------- | ------------------ |
| `biblio.gmd_id`           | `mst_gmd`          |
| `biblio.publisher_id`     | `mst_publisher`    |
| `biblio.language_id`      | `mst_language`     |
| `biblio.publish_place_id` | `mst_place`        |
| `item.coll_type_id`       | `mst_coll_type`    |
| `item.location_id`        | `mst_location`     |
| `item.item_status_id`     | `mst_item_status`  |
| `member.member_type_id`   | `mst_member_type`  |
| `loan.loan_rules_id`      | `mst_loan_rules`   |
| `biblio.content_type_id`  | `mst_content_type` |
| `biblio.media_type_id`    | `mst_media_type`   |
| `biblio.carrier_type_id`  | `mst_carrier_type` |

## Schema Features

| Feature          | Detail                                                                                                                                                           |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Extension tables | `biblio_custom`, `item_custom`, `member_custom`, `mst_custom_field` for custom fields beyond default schema.                                                     |
| System config    | `setting` stores key-value pairs. `group_access` has per-group module permissions + JSON `menus`. `user.groups` is serialized text, not a normalized join table. |
| Search/index     | `search_biblio` is the main flattened helper. `index_words` + `index_documents` for IR internals. `FULLTEXT` indexes on several tables.                          |
| Logs/history     | `loan_history` (circulation traces), `biblio_log`, `system_log`, `backup_log`, `visitor_count`, `files_read` (operational activity).                             |

## Reading Order

1. Catalog: `biblio` -> `item` -> `biblio_author` / `biblio_topic` -> `mst_*`
2. Circulation: `member` -> `loan` -> `loan_history` -> `reserve` / `fines` -> `mst_loan_rules`
3. OPAC/search: `biblio` -> `search_biblio` -> `files` / `biblio_attachment` -> `comment`
4. Admin: `user` -> `user_group` -> `group_access` -> `mst_module` -> `setting`
5. Serials: `serial` -> `kardex` -> `mst_frequency`

## Caveats

- `MyISAM` means FK-style references exist only by naming convention and indexes.
- `item_code` and `member_id` are operational keys appearing directly in transaction tables.
- Some business rules are encoded in serialized fields (`mst_item_status.rules`, `setting.setting_value`).
- A live SLiMS instance may diverge from `senayan.sql` — use this file as baseline and compare explicitly.
