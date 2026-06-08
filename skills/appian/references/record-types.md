# Record Types

## CLI Commands

```bash
# Create a record type
appian rt create --app $APP --file case.json

# List record types in an application
appian rt list --app $APP

# Get record type details (includes fields, source config, typeReference)
appian rt get <uuid>

# Update a record type
appian rt get $UUID | jq '.description = "Updated"' | appian rt update $UUID

# Delete a record type
appian rt delete <uuid>

# List fields
appian rt fields list <uuid>

# Add a field
echo '{"fieldName":"status","fieldType":"TEXT","length":50}' | appian rt fields add <uuid>

# List relationships
appian rt relationships list <uuid>

# List views
appian rt views list <uuid>

# List actions
appian rt actions list <uuid>

# List user filters
appian rt filters list <uuid>
```

## Create JSON Schema

```json
{
  "name": "Case",
  "pluralName": "Cases",
  "sourceType": "DATABASE",
  "createTable": true,
  "description": "Support cases submitted by customers",
  "fields": [
    {"fieldName": "case_id", "fieldType": "INTEGER", "isPrimaryKey": true},
    {"fieldName": "title", "fieldType": "TEXT", "length": 255},
    {"fieldName": "description", "fieldType": "TEXT", "length": 4000},
    {"fieldName": "status", "fieldType": "TEXT", "length": 50},
    {"fieldName": "priority", "fieldType": "INTEGER"},
    {"fieldName": "assigned_username", "fieldType": "USER"},
    {"fieldName": "customer_id", "fieldType": "INTEGER"},
    {"fieldName": "created_by_username", "fieldType": "USER"},
    {"fieldName": "modified_by_username", "fieldType": "USER"},
    {"fieldName": "created_at", "fieldType": "DATETIME"},
    {"fieldName": "updated_at", "fieldType": "DATETIME"}
  ]
}
```

## Source Configuration

- `sourceType`: `DATABASE` (most common), `WEB_SERVICE`, `PROCESS`, `SALESFORCE`
- `createTable`: `true` — Appian creates the DB table. `false` — table must already exist.
- `tableName`: UPPER_SNAKE_CASE, singular. Defaults to snake_case of record type name.
- `dataSourceUuid` and `schema`: for DATABASE types. Discover from existing record types in the same app if available.

## Field Configuration

Each field requires:
- `fieldName` — snake_case column name (Appian auto-converts to camelCase for the record field)
- `fieldType` — one of: TEXT, NUMBER, INTEGER, DECIMAL, DATE, DATETIME, TIME, BOOLEAN, USER, GROUP, DOCUMENT, FOLDER

Optional:
- `isPrimaryKey` — exactly one per record type, must be INTEGER
- `isUnique` — unique constraint
- `length` — for TEXT fields (0 = unlimited)
- `displayName`, `description`

Load `references/field-types.md` for the complete field type reference.

## Naming Conventions

- **Record type name**: `PREFIX EntityName` in Title Case, singular — `CM Case`, `CM Customer`, `CM Letter of Authorization`
- **Plural name**: Title Case, plural, no prefix — `Cases`, `Customers`, `Letters of Authorization`
- **Table name**: UPPER_SNAKE_CASE, singular — `CASE`, `CUSTOMER`, `LETTER_OF_AUTHORIZATION`
- **Field names**: snake_case — `case_id`, `created_at`, `assigned_username`
- **Primary key**: `[table_name_lowercase]_id` — table CASE → `case_id`
- **Foreign keys**: `[referenced_table_lowercase]_id` — referencing CUSTOMER → `customer_id`

## Standard Field Structure

Order: primary key → business fields → foreign keys → audit fields.

1. Surrogate PK: `[table]_id` (INTEGER, isPrimaryKey: true)
2. Business fields from requirements
3. Foreign key fields (INTEGER)
4. Audit fields (entities only, not reference/junction tables):
   - `created_by_username` (USER)
   - `modified_by_username` (USER)
   - `created_at` (DATETIME)
   - `updated_at` (DATETIME)

## Relationships

Relationships require both record types to exist first. Add them after creating all record types.

```bash
# Add a relationship (JSON via stdin)
echo '{
  "relationshipName": "customer",
  "sourceRecordTypeFieldUuid": "<fk-field-uuid>",
  "targetRecordTypeFieldUuid": "<pk-field-uuid>",
  "targetRecordTypeUuid": "<target-rt-uuid>",
  "relationshipType": "MANY_TO_ONE"
}' | appian rt relationships add <source-rt-uuid>
```

### Bidirectional Requirement

Always declare both sides:
- FK table: `MANY_TO_ONE` (or `ONE_TO_ONE` with unique FK)
- Referenced table: `ONE_TO_MANY` (or `ONE_TO_ONE`)

A one-sided relationship breaks record type traversal.

### Relationship Naming

- **MANY_TO_ONE**: remove `_id` from FK field, camelCase — `customer_id` → `customer`
- **ONE_TO_MANY to entities**: pluralize target in camelCase — `CASE_NOTE` → `caseNotes`
- **ONE_TO_MANY to junction**: pluralize the other entity — CASE→CASE_TAG→TAG: name is `tags`
- **ONE_TO_ONE**: same as MANY_TO_ONE on FK side; entity name on other side

### Relationship Types

| Pattern | Implementation |
|---|---|
| One-to-Many | "many" holds FK (MANY_TO_ONE); "one" declares ONE_TO_MANY |
| Many-to-Many | Junction table with two FKs and two MANY_TO_ONE; both entities declare ONE_TO_MANY to junction |
| Self-Referential | Both directions on same RT (e.g., `parent_case_id` → parentCase / childCases) |

## Updating Record Types

**Always get before update** — updates replace provided fields entirely.

```bash
# Add a field to existing RT
echo '{"fieldName":"priority","fieldType":"INTEGER"}' | appian rt fields add $RT_UUID

# Update RT metadata
appian rt get $UUID | jq '.description = "New desc"' | appian rt update $UUID
```

## Common Pitfalls

- **Creating relationships before all RTs exist** — both source and target must exist first
- **One-sided relationships** — always declare both MANY_TO_ONE + ONE_TO_MANY
- **Composite primary keys** — Appian requires single-column INTEGER surrogate PK
- **Missing `createTable: true`** — without it, Appian expects table to already exist
- **Wrong field type names** — case-sensitive: TEXT, INTEGER, DATETIME (not text, integer)
- **Audit fields on junction/reference tables** — only entity tables get audit fields
- **Not discovering existing RTs** — always `appian rt list --app $APP` to check what exists and get dataSourceUuid/schema from existing RTs
