# DSpace 6.x REST API Reference

## Identity & Global Rules

| Property  | Value                                                |
| --------- | ---------------------------------------------------- |
| Base path | `/rest`                                              |
| Formats   | `application/json` (default), `application/xml`      |
| Legacy    | DSpace 6 support ended July 1, 2023                  |
| Version   | 6.x only — not applicable to DSpace 7+ `/server/api` |

| Concern     | Rule                                                                                     |
| ----------- | ---------------------------------------------------------------------------------------- |
| Pagination  | `limit` + `offset` on list endpoints. Defaults: `offset=0`, `limit=100`.                 |
| Total count | Not available in paginated responses.                                                    |
| Expands     | `?expand=parentCollection,bitstreams` for targeted; `?expand=all` for inspection only.   |
| Writes      | Send object body matching the endpoint's documented type.                                |
| Local SSL   | Comment out `security-constraint` in `web.xml` for localhost only. Production keeps SSL. |

## Authentication

| Method | Endpoint  | Behavior                                                             |
| ------ | --------- | -------------------------------------------------------------------- |
| `GET`  | `/`       | Endpoint index.                                                      |
| `POST` | `/login`  | Body: `email=...&password=...`. Returns `Set-Cookie: JSESSIONID=...` |
| `GET`  | `/status` | Current auth state + version. Returns Status object.                 |
| `POST` | `/logout` | Invalidate `JSESSIONID`.                                             |
| `GET`  | `/test`   | Returns `REST api is running`.                                       |

- Auth is `JSESSIONID` cookie-based. The older `rest-dspace-token` scheme is not supported in 6.x.
- URL-encode special characters in credentials and query strings.
- Invalid login returns `401`.
- Shibboleth: source PDF references both `/shibbolethlogin` and `/shibboleth-login`. Verify against deployed config.

## Communities

| Method   | Endpoint                                  | Behavior                           |
| -------- | ----------------------------------------- | ---------------------------------- |
| `GET`    | `/communities`                            | List all. Paginated.               |
| `GET`    | `/communities/top-communities`            | Top-level only. Paginated.         |
| `GET`    | `/communities/{id}`                       | Get one.                           |
| `GET`    | `/communities/{id}/collections`           | Child collections. Paginated.      |
| `GET`    | `/communities/{id}/communities`           | Sub-communities. Paginated.        |
| `POST`   | `/communities`                            | Create top-level. Body: Community. |
| `POST`   | `/communities/{id}/collections`           | Create collection under community. |
| `POST`   | `/communities/{id}/communities`           | Create sub-community.              |
| `PUT`    | `/communities/{id}`                       | Update. Body: Community.           |
| `DELETE` | `/communities/{id}`                       | Delete.                            |
| `DELETE` | `/communities/{id}/collections/{coll-id}` | Remove collection from community.  |
| `DELETE` | `/communities/{id}/communities/{sub-id}`  | Remove sub-community.              |

## Collections

| Method   | Endpoint                            | Behavior                                          |
| -------- | ----------------------------------- | ------------------------------------------------- |
| `GET`    | `/collections`                      | List all. Paginated.                              |
| `GET`    | `/collections/{id}`                 | Get one.                                          |
| `GET`    | `/collections/{id}/items`           | List items. Paginated.                            |
| `POST`   | `/collections/{id}/items`           | Create item. Body: Item.                          |
| `POST`   | `/collections/find-collection`      | Find by exact name. Body: plain string, not JSON. |
| `PUT`    | `/collections/{id}`                 | Update. Body: Collection.                         |
| `DELETE` | `/collections/{id}`                 | Delete.                                           |
| `DELETE` | `/collections/{id}/items/{item-id}` | Remove item from collection.                      |

## Items

| Method   | Endpoint                             | Behavior                                                                                            |
| -------- | ------------------------------------ | --------------------------------------------------------------------------------------------------- |
| `GET`    | `/items`                             | List all. Paginated.                                                                                |
| `GET`    | `/items/{id}`                        | Get one.                                                                                            |
| `GET`    | `/items/{id}/metadata`               | Get metadata.                                                                                       |
| `GET`    | `/items/{id}/bitstreams`             | List bitstreams. Paginated.                                                                         |
| `POST`   | `/items/find-bymetadata-field`       | Find by MetadataEntry.                                                                              |
| `POST`   | `/items/{id}/metadata`               | Add metadata. Body: `[]MetadataEntry`.                                                              |
| `POST`   | `/items/{id}/bitstreams?name={file}` | Upload bitstream. Body: raw file bytes. Optional: `description`, `groupId`, `year`, `month`, `day`. |
| `PUT`    | `/items/{id}/metadata`               | Replace matching metadata entries. Body: MetadataEntry.                                             |
| `DELETE` | `/items/{id}`                        | Delete.                                                                                             |
| `DELETE` | `/items/{id}/metadata`               | Clear all metadata.                                                                                 |
| `DELETE` | `/items/{id}/bitstreams/{bs-id}`     | Remove bitstream.                                                                                   |

Bitstream upload:

```bash
curl -X POST --cookie "JSESSIONID=..." --data-binary @paper.pdf \
  "https://dspace.example.edu/rest/items/14301/bitstreams?name=paper.pdf&description=Main%20file"
```

## Bitstreams

| Method   | Endpoint                              | Behavior                                     |
| -------- | ------------------------------------- | -------------------------------------------- |
| `GET`    | `/bitstreams`                         | List all. Paginated.                         |
| `GET`    | `/bitstreams/{id}`                    | Get one. `?expand=parent` for parent object. |
| `GET`    | `/bitstreams/{id}/policy`             | List resource policies.                      |
| `GET`    | `/bitstreams/{id}/retrieve`           | Download file data.                          |
| `POST`   | `/bitstreams/{id}/policy`             | Add policy. Body: ResourcePolicy.            |
| `PUT`    | `/bitstreams/{id}/data`               | Replace file bytes.                          |
| `PUT`    | `/bitstreams/{id}`                    | Update metadata only. Body: Bitstream.       |
| `DELETE` | `/bitstreams/{id}`                    | Delete.                                      |
| `DELETE` | `/bitstreams/{id}/policy/{policy-id}` | Delete policy.                               |

## Handle & Hierarchy

| Method | Endpoint                    | Behavior                                          |
| ------ | --------------------------- | ------------------------------------------------- |
| `GET`  | `/handle/{prefix}/{suffix}` | Resolve Community, Collection, or Item by handle. |
| `GET`  | `/hierarchy`                | Lightweight nested community/collection tree.     |

## Registries

| Method   | Endpoint                           | Behavior                                                              |
| -------- | ---------------------------------- | --------------------------------------------------------------------- |
| `GET`    | `/registries/schema`               | List schemas.                                                         |
| `GET`    | `/registries/schema/{prefix}`      | Get one schema.                                                       |
| `POST`   | `/registries/schema`               | Create schema. Body: Schema.                                          |
| `DELETE` | `/registries/schema/{id}`          | Delete schema. `PUT` is not implemented (schema has no mutable data). |
| `GET`    | `/registries/metadata-fields`      | List fields.                                                          |
| `GET`    | `/registries/metadata-fields/{id}` | Get one field.                                                        |
| `POST`   | `/registries/metadata-fields`      | Create field. Body: MetadataField.                                    |
| `PUT`    | `/registries/metadata-fields/{id}` | Update field.                                                         |
| `DELETE` | `/registries/metadata-fields/{id}` | Delete field.                                                         |

## Reports

| Method | Endpoint                     | Behavior                                         |
| ------ | ---------------------------- | ------------------------------------------------ |
| `GET`  | `/reports`                   | List report tools.                               |
| `GET`  | `/reports/{nickname}`        | Redirect to report.                              |
| `GET`  | `/filtered-collections`      | Collections + item counts by predefined filters. |
| `GET`  | `/filtered-collections/{id}` | Filtered items + counts for one collection.      |
| `GET`  | `/filtered-items`            | Items by metadata query + filters.               |

## Object Reference

### Community

| Field            | Type           | Note                                                              |
| ---------------- | -------------- | ----------------------------------------------------------------- |
| id               | int            |                                                                   |
| name             | string         |                                                                   |
| handle           | string         |                                                                   |
| type             | string         | `"community"`                                                     |
| link             | string         | Self-link                                                         |
| expand           | []string       | `parentCommunity`, `collections`, `subCommunities`, `logo`, `all` |
| logo             | Bitstream/null |                                                                   |
| parentCommunity  | Community/null |                                                                   |
| copyrightText    | string         |                                                                   |
| introductoryText | string         |                                                                   |
| shortDescription | string         |                                                                   |
| sidebarText      | string         |                                                                   |
| countItems       | int            |                                                                   |
| subcommunities   | []Community    |                                                                   |
| collections      | []Collection   |                                                                   |

### Collection

| Field               | Type           | Note                                                                        |
| ------------------- | -------------- | --------------------------------------------------------------------------- |
| id                  | int            |                                                                             |
| name                | string         |                                                                             |
| handle              | string         |                                                                             |
| type                | string         | `"collection"`                                                              |
| link                | string         | Self-link                                                                   |
| expand              | []string       | `parentCommunityList`, `parentCommunity`, `items`, `license`, `logo`, `all` |
| logo                | Bitstream/null |                                                                             |
| parentCommunity     | Community/null |                                                                             |
| parentCommunityList | []Community    |                                                                             |
| items               | []Item         |                                                                             |
| license             | string/null    |                                                                             |
| copyrightText       | string         |                                                                             |
| introductoryText    | string         |                                                                             |
| shortDescription    | string         |                                                                             |
| sidebarText         | string         |                                                                             |
| numberItems         | int            |                                                                             |

### Item

| Field                | Type              | Note                                                                                               |
| -------------------- | ----------------- | -------------------------------------------------------------------------------------------------- |
| id                   | int               |                                                                                                    |
| name                 | string            |                                                                                                    |
| handle               | string            |                                                                                                    |
| type                 | string            | `"item"`                                                                                           |
| link                 | string            | Self-link                                                                                          |
| expand               | []string          | `metadata`, `parentCollection`, `parentCollectionList`, `parentCommunityList`, `bitstreams`, `all` |
| lastModified         | string            | Format: `"2015-01-12 15:44:12.978"`                                                                |
| parentCollection     | Collection/null   |                                                                                                    |
| parentCollectionList | []Collection/null |                                                                                                    |
| parentCommunityList  | []Community/null  |                                                                                                    |
| bitstreams           | []Bitstream/null  |                                                                                                    |
| archived             | string            | `"true"` / `"false"`                                                                               |
| withdrawn            | string            | `"true"` / `"false"`                                                                               |

### Bitstream

| Field        | Type                  | Note                           |
| ------------ | --------------------- | ------------------------------ |
| id           | int                   |                                |
| name         | string                |                                |
| handle       | null                  | Always null for bitstreams     |
| type         | string                | `"bitstream"`                  |
| link         | string                | Self-link                      |
| expand       | []string              | `parent`, `policies`, `all`    |
| bundleName   | string                | e.g., `"ORIGINAL"`             |
| description  | string                |                                |
| format       | string                | e.g., `"Adobe PDF"`            |
| mimeType     | string                |                                |
| sizeBytes    | int                   |                                |
| parentObject | object/null           |                                |
| retrieveLink | string                | Download path                  |
| checkSum     | object                | `{ value, checkSumAlgorithm }` |
| sequenceId   | int                   |                                |
| policies     | []ResourcePolicy/null |                                |

### Compact Objects

**ResourcePolicy**: `{ id, action, epersonId, groupId, resourceId, resourceType, rpDescription, rpName, rpType, startDate, endDate }` — `rpType` values include `TYPE_INHERITED`.

**MetadataEntry**: `{ key, value, language }` — `key` uses dotted notation (`dc.description.abstract`). `language` is nullable.

**Schema**: `{ namespace, prefix }`

**MetadataField**: `{ description, element, name, qualifier }` — `name` format: `prefix.element`. `qualifier` is nullable.

**User**: `{ email, password }`

**Status**: `{ okay, authenticated, email, fullname, token }` — `token` field appears in responses but 6.x uses `JSESSIONID` cookie auth, not token-based.
