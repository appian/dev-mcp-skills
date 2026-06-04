---
name: "appian-record-types"
description: "Create and modify Appian record types including fields, relationships, and source configuration. Covers field types, naming conventions, relationship management, and database table creation. Use when working with record types, data models, database tables, or record type relationships."
---

## Relevant Tools

Record type tools cover the full lifecycle:

- **Create a record type** — provide a name, source configuration (with fields and database table settings), and optionally a plural name, description, and relationships
- **List and get record types** — retrieve record types by UUID or search with optional filtering; scope to an application with `appUuid`
- **Update a record type** — modify name, description, plural name, source configuration, or relationships on an existing record type
- **Delete a record type** — permanently remove a record type by UUID
- **List and get fields** — retrieve all fields or a specific field by name for a record type
- **List and add relationships** — retrieve existing relationships or add new ones to a record type
- **Get source configuration** — retrieve the current source configuration including data source, table name, schema, and fields

## Creating Record Types

### Source Configuration

Every record type needs a source configuration that defines where its data lives. For database-backed record types (the most common case):

- Set `sourceType` to `DATABASE`
- Set `createTable` to `true` to have Appian create the database table automatically
- Provide a `tableName` in UPPER_SNAKE_CASE (singular noun, e.g., `CASE`, `CUSTOMER`, `LETTER_OF_AUTHORIZATION`)
- Provide `dataSourceUuid` and `schema` from the target database — discover these from existing record types if available
- Define fields in `sourceAndCustomFields`

### Field Configuration

Each field in `sourceAndCustomFields` requires:

- `fieldName` — snake_case column name (Appian auto-converts to camelCase for the record type field)
- `fieldType` — one of the 12 valid types (see field-types reference)
- `isPrimaryKey` — exactly one field per record type must be `true`; use an INTEGER auto-increment surrogate key
- Optional: `displayName`, `description`, `isUnique`, `length` (for TEXT fields)

Load `references/field-types.md` for the complete field type reference with constraints and usage guidance.

### Naming Conventions

- **Record type name**: Title Case, singular (e.g., "Case", "Customer", "Letter of Authorization")
- **Plural name**: Title Case, plural (e.g., "Cases", "Customers", "Letters of Authorization")
- **Table name**: UPPER_SNAKE_CASE, singular (e.g., `CASE`, `CUSTOMER`, `LETTER_OF_AUTHORIZATION`)
- **Field names**: snake_case (e.g., `case_id`, `created_at`, `assigned_username`)
- **Primary key**: `[table_name_lowercase]_id` (e.g., table CASE → `case_id`)
- **Foreign keys**: `[referenced_table_lowercase]_id` (e.g., referencing CASE_STATUS → `case_status_id`)

### Standard Field Structure

Order fields consistently: primary key → business fields → foreign keys → audit fields.

1. Surrogate primary key: `[table_name_lowercase]_id` (INTEGER, isPrimaryKey: true)
2. Business fields from requirements
3. Foreign key fields to related record types (INTEGER)
4. Audit fields on entities (not on reference or junction tables):
   - `created_by_username` (USER)
   - `modified_by_username` (USER)
   - `created_at` (DATETIME)
   - `updated_at` (DATETIME)

## Managing Relationships

### Operation Sequence

Relationships require both record types to exist before they can be linked. Follow this order:

1. Create all record types with their fields and source configuration first
2. After all record types exist, add relationships between them
3. For cross-type relationships, you need the field UUIDs from both sides — retrieve them by listing the fields on each record type

### Adding Relationships

Each relationship requires:

- `relationshipName` — camelCase name following the naming algorithm below
- `sourceRecordTypeFieldUuid` — UUID of the foreign key field on the source record type
- `targetRecordTypeFieldUuid` — UUID of the referenced field (usually the primary key) on the target record type
- `targetRecordTypeUuid` — UUID of the target record type
- `relationshipType` — one of: `ONE_TO_MANY`, `MANY_TO_ONE`, `ONE_TO_ONE`

### Bidirectional Relationships

All relationships must be declared on both sides for Appian record type traversal to work:

- The table with the FK field declares `MANY_TO_ONE` (or `ONE_TO_ONE` with a unique FK)
- The referenced table declares `ONE_TO_MANY` (or `ONE_TO_ONE`)
- Always add both sides — a one-sided relationship breaks record type traversal

### Relationship Naming

Derive relationship names algorithmically for consistency:

- **MANY_TO_ONE**: remove `_id` from FK field name, convert to camelCase (`customer_id` → `customer`, `case_status_id` → `caseStatus`)
- **ONE_TO_MANY to entities**: pluralize target entity name in camelCase (`CASE_NOTE` → `caseNotes`)
- **ONE_TO_MANY to junction tables**: pluralize the other connected entity in camelCase (CASE → CASE_TAG → TAG: name is `tags`)
- **ONE_TO_ONE**: same as MANY_TO_ONE on the FK side; target entity name in camelCase on the other side

### Relationship Patterns

- **One-to-Many**: the "many" side holds the FK field and declares MANY_TO_ONE; the "one" side declares ONE_TO_MANY
- **Many-to-Many**: implement via a junction table — the junction table has two FK fields and two MANY_TO_ONE relationships; both connected entities declare ONE_TO_MANY to the junction table
- **Self-Referential**: both MANY_TO_ONE and ONE_TO_MANY declared on the same record type (e.g., `parent_case_id` → parentCase / childCases)

## Updating Record Types

### Read Before Update

Always retrieve the current state of a record type before updating it. The update operation replaces the provided fields entirely — if you update `sourceConfiguration` without including existing fields, they will be lost.

1. Get the record type to see its current configuration
2. Get the source configuration to see current fields
3. Merge your changes with the existing state
4. Send the complete updated configuration

### Adding Fields to an Existing Record Type

To add fields to an existing record type, retrieve the current source configuration, append the new fields to the existing `sourceAndCustomFields` list, and update the record type with the complete field list.

### Modifying Fields

To modify a field, retrieve the current source configuration, find the field in `sourceAndCustomFields`, change the desired properties, and update with the complete field list.

## Common Pitfalls

- **Forgetting to create all record types before adding relationships** — relationships require both source and target record types to exist; create all record types first, then add relationships in a second pass
- **Adding a relationship on only one side** — always declare both sides (MANY_TO_ONE + ONE_TO_MANY) for bidirectional traversal
- **Using composite primary keys** — Appian requires a single-column INTEGER surrogate primary key per record type
- **Updating source configuration without existing fields** — the update replaces the entire source configuration; always read current state first and merge
- **Missing the `createTable: true` flag** — without it, Appian expects the database table to already exist
- **Using wrong field type names** — field types are case-sensitive; use exact values: TEXT, NUMBER, INTEGER, DECIMAL, DATE, DATETIME, TIME, BOOLEAN, USER, GROUP, DOCUMENT, FOLDER
- **Forgetting `length` on TEXT fields** — TEXT fields default to length 0 (unlimited); set an appropriate length for database column sizing
- **Adding audit fields to reference or junction tables** — only entity tables get audit fields (created_by, modified_by, created_at, updated_at)
- **Creating user/role record types** — use USER field type referencing Appian's built-in user system instead of creating custom user entities
- **Not discovering existing record types before creating** — always check what already exists in the application to avoid duplicates and to discover data source configuration (dataSourceUuid, schema) from existing record types

## Data Modeling Guidance

Record type design follows relational data modeling principles. For guidance on normalization, entity vs. reference table classification, junction table patterns, and schema validation, activate the data modeling knowledge skill. This skill focuses on the record type object lifecycle and platform mechanics; data modeling covers the design discipline.

## When You Need More

For questions about specific Appian data types, advanced record type configuration, or platform behavior beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.

Load `references/field-types.md` when you need the complete field type reference with constraints and usage guidance.
