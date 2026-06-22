# Record Types

## Create JSON Schema

```json
{
  "name": "CM Case",
  "pluralName": "Cases",
  "sourceType": "DATABASE",
  "createTable": true,
  "description": "Support cases submitted by customers",
  "fields": [
    {"fieldName": "caseId", "fieldType": "INTEGER", "isPrimaryKey": true},
    {"fieldName": "title", "fieldType": "TEXT", "length": 255},
    {"fieldName": "description", "fieldType": "TEXT", "length": 4000},
    {"fieldName": "status", "fieldType": "TEXT", "length": 50},
    {"fieldName": "priority", "fieldType": "INTEGER"},
    {"fieldName": "assignedUsername", "fieldType": "USER"},
    {"fieldName": "customerId", "fieldType": "INTEGER"},
    {"fieldName": "createdByUsername", "fieldType": "USER"},
    {"fieldName": "modifiedByUsername", "fieldType": "USER"},
    {"fieldName": "createdAt", "fieldType": "DATETIME"},
    {"fieldName": "updatedAt", "fieldType": "DATETIME"}
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
- `fieldName` — camelCase identifier used in SAIL expressions (e.g., `firstName`, `caseId`). The platform auto-generates the database column name in UPPER_SNAKE_CASE.
- `fieldType` — one of: TEXT, NUMBER, INTEGER, DECIMAL, DATE, DATETIME, TIME, BOOLEAN, USER, GROUP, DOCUMENT, FOLDER

Optional:
- `isPrimaryKey` — exactly one per record type, must be INTEGER
- `isUnique` — unique constraint
- `length` — for TEXT fields (0 = unlimited)
- `displayName`, `description`

Load `references/field-types.md` for the complete field type reference.

### USER Field Requirements

**When creating USER type fields, you MUST add relationships to SYSTEM_RECORD_TYPE_USER.** This is a separate step performed AFTER record type creation.

USER fields (e.g., `createdBy`, `modifiedBy`, `assignedTo`) store usernames but require an explicit MANY_TO_ONE relationship to the system User record type for full functionality. Without this relationship:
- USER fields display as plain text (no name, email, profile picture)
- Related records queries fail
- Record views cannot navigate to user details
- Relationship traversal breaks in interfaces

**See `references/relationship-patterns.md` → "USER Field Relationships to SYSTEM_RECORD_TYPE_USER" for the complete pattern including:**
- Required relationship configuration
- Platform constants (SYSTEM_RECORD_TYPE_USER, SYSTEM_RECORD_TYPE_USER_FIELD_username)
- Naming conventions
- Execution order

This relationship is mandatory for every USER field in every record type.

## Naming Conventions

- **Record type name**: `PREFIX EntityName` in Title Case, singular — `CM Case`, `CM Customer`, `CM Letter of Authorization`
- **Plural name**: Title Case, plural, no prefix — `Cases`, `Customers`, `Letters of Authorization`
- **Table name**: UPPER_SNAKE_CASE, singular — `CASE`, `CUSTOMER`, `LETTER_OF_AUTHORIZATION`
- **Field names**: camelCase — `caseId`, `createdAt`, `assignedUsername` (DB column auto-generated as UPPER_SNAKE_CASE)
- **Primary key**: `[entityName]Id` in camelCase — `caseId`, `employeeId`
- **Foreign keys**: `[referencedEntity]Id` in camelCase — `customerId`, `caseStatusId`

## Standard Field Structure

Order: primary key → business fields → foreign keys → audit fields.

1. Surrogate PK: `[table]_id` (INTEGER, isPrimaryKey: true)
2. Business fields from requirements
3. Foreign key fields (INTEGER)
4. Audit fields (entities only, not reference/junction tables):
   - `createdByUsername` (USER)
   - `modifiedByUsername` (USER)
   - `createdAt` (DATETIME)
   - `updatedAt` (DATETIME)

## Relationships

Relationships require both record types to exist first. Add them after creating all record types.

### Relationship JSON Schema

```json
{
  "relationshipName": "customer",
  "sourceRecordTypeFieldUuid": "<fk-field-uuid>",
  "targetRecordTypeFieldUuid": "<pk-field-uuid>",
  "targetRecordTypeUuid": "<target-rt-uuid>",
  "relationshipType": "MANY_TO_ONE"
}
```

### Bidirectional Requirement

Always declare both sides:
- FK table: `MANY_TO_ONE` (or `ONE_TO_ONE` with unique FK)
- Referenced table: `ONE_TO_MANY` (or `ONE_TO_ONE`)

A one-sided relationship breaks record type traversal.

### Relationship Naming

- **MANY_TO_ONE**: remove `Id` suffix, keep camelCase — `customerId` → `customer`, `caseStatusId` → `caseStatus`
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

**Always get before update** — updates replace provided fields entirely. Get the current state, modify what you need, then update.

## Record Actions

Record actions are process-model-backed operations surfaced in the record list (LIST_ACTION) or on individual record rows (RELATED_ACTION).

### JSON Schema

```json
{
  "displayName": "Edit Submission",
  "processModelUuid": "<pm-uuid>",
  "actionType": "RELATED_ACTION",
  "key": "editSubmission",
  "visibilityExpr": "=loggedInUser() = rv!record['recordType!{rt-uuid}Submission.fields.{fid}submittedByUsername']",
  "contextExpr": "={record: rv!record}",
  "icon": "f044",
  "dialogWidth": "MEDIUM_PLUS"
}
```

| Field | Required | Description |
|---|---|---|
| `displayName` | Yes | Label shown to users |
| `processModelUuid` | Yes | Process model to launch |
| `actionType` | Yes | `LIST_ACTION` (record list button) or `RELATED_ACTION` (per-row action) |
| `key` | Yes | Unique key for SAIL references (e.g., `recordType!Case.actions.editCase`) |
| `visibilityExpr` | No | SAIL expression → `true` shows the action, `false` hides it. Default: `=true()` (visible to all). Uses `rv!record` for field access. |
| `contextExpr` | No | SAIL expression passing record data to the process model. RELATED_ACTION only. Keys map to process parameter names. |
| `icon` | No | Font Awesome hex code (e.g., `f044` = pencil, `f1f8` = trash, `f067` = plus) |
| `dialogWidth` | No | `NARROW`, `MEDIUM`, `MEDIUM_PLUS` (default), `WIDE`, `EXTRA_WIDE`, `FIT` |
| `dialogHeight` | No | `AUTO` (default), `FIT`, `SHORT`, `MEDIUM`, `TALL`, `EXTRA_TALL` |

### Action Types

| Type | Where it appears | `rv!record` available | Typical use |
|---|---|---|---|
| `LIST_ACTION` | Button on the record list header | No | Create new records |
| `RELATED_ACTION` | Per-row action on record views/grids | Yes | Edit, delete, status transitions |

### Visibility Expressions

Control who sees the action. Omitting `visibilityExpr` or setting `"=true()"` makes the action visible to all users.

**Owner-only (record owner can edit):**
```
"visibilityExpr": "=loggedInUser() = rv!record['recordType!{rt-uuid}Entity.fields.{fid}createdByUsername']"
```

**Group-restricted (only admins):**
```
"visibilityExpr": "=a!isUserMemberOfGroup(loggedInUser(), cons!MY_ADMIN_GROUP)"
```

**Status-conditional (only when open):**
```
"visibilityExpr": "=rv!record['recordType!{rt-uuid}Case.fields.{fid}status'] = \"Open\""
```

**Combined (owner AND open status):**
```
"visibilityExpr": "=and(loggedInUser() = rv!record['recordType!{rt-uuid}Case.fields.{fid}assignedTo'], rv!record['recordType!{rt-uuid}Case.fields.{fid}status'] = \"Open\")"
```

### Context Expressions

For RELATED_ACTION, `contextExpr` passes the record into the process model. The keys in the dictionary must match process parameter names (case-sensitive):

```
"contextExpr": "={record: rv!record}"
```

The process model needs a parameter named `record` typed to the record type's `typeReference`.

### Examples

**LIST_ACTION — create new (no visibility restriction needed):**
```json
{
  "displayName": "New Submission",
  "processModelUuid": "<create-pm-uuid>",
  "actionType": "LIST_ACTION",
  "key": "createSubmission",
  "icon": "f067"
}
```

**RELATED_ACTION — edit, restricted to record owner:**
```json
{
  "displayName": "Edit Submission",
  "processModelUuid": "<update-pm-uuid>",
  "actionType": "RELATED_ACTION",
  "key": "editSubmission",
  "contextExpr": "={record: rv!record}",
  "visibilityExpr": "=loggedInUser() = rv!record['recordType!{rt-uuid}Submission.fields.{fid}submittedByUsername']",
  "icon": "f044"
}
```

**RELATED_ACTION — delete, restricted to admins:**
```json
{
  "displayName": "Delete",
  "processModelUuid": "<delete-pm-uuid>",
  "actionType": "RELATED_ACTION",
  "key": "deleteSubmission",
  "visibilityExpr": "=a!isUserMemberOfGroup(loggedInUser(), cons!MY_ADMIN_GROUP)",
  "icon": "f1f8"
}
```

### Design Guidance

- **LIST_ACTION** (create new): typically no visibility restriction — any user who can see the record list can create
- **RELATED_ACTION** (edit/update): restrict to record owner via `visibilityExpr` comparing `loggedInUser()` to the owner/submitter field
- **RELATED_ACTION** (delete): restrict to admins or record owner
- Always set `contextExpr` on RELATED_ACTION to pass the record into the process model
- The `key` must be unique within the record type and is used in SAIL references
- RELATED_ACTIONs auto-surface only on record views, NOT on custom `a!gridField` grids. When adding a RELATED_ACTION (e.g., Edit), also update any dashboard interface that displays that record type in a custom grid — add `recordActions` to the grid or a link column using `a!recordActionField()` to surface the action on each row.

## User Filters

User filters let end users filter the record list via dropdowns or date pickers. Three types: `LIST_OF_VALUES`, `DATE_RANGE`, `EXPRESSION`.

Add filters after record type fields and relationships exist (step 10 in dependency order). Load `references/record-type-user-filters.md` for full schemas, EXPRESSION filter examples, related record filtering, and design guidance.

## Common Pitfalls

- **Creating relationships before all RTs exist** — both source and target must exist first
- **One-sided relationships** — always declare both MANY_TO_ONE + ONE_TO_MANY
- **Composite primary keys** — Appian requires single-column INTEGER surrogate PK named `id`
- **Missing `createTable: true`** — without it, Appian expects table to already exist
- **Wrong field type names** — case-sensitive: TEXT, INTEGER, DATETIME (not text, integer)
- **Audit fields on junction/reference tables** — only entity tables get audit fields
- **Not discovering existing RTs** — always list record types in the app to check what exists and get dataSourceUuid/schema from existing RTs
- **Version conflicts on relationships** — add relationships sequentially, not in parallel (each add increments versionId)
- **Wrong title expression format** — use `rv!record['recordType!{uuid}Name.fields.{fieldUuid}fieldName']`, not `rf!field`
- **Fabricating usernames in sample data** — query SYSTEM_RECORD_TYPE_USER for real usernames
- **PK named `caseId` or `customerId`** — must be just `id`
- **USER fields with Username suffix** — use `createdBy`, `modifiedBy` (not `createdByUsername`)

## Title Expression

Set a title expression after creating a record type to define the text shown in record links and headers.

**Format:**
```
rv!record['recordType!{recordTypeUuid}RecordTypeName.fields.{fieldUuid}fieldName']
```

**Construction:**
1. Get the record type UUID and exact name (including spaces) from the creation response
2. Get the field UUID for the title field (`label` for reference tables, `title`/`name` for entities)
3. Combine using the exact format above — record type name must include spaces, field name is camelCase

**Common mistakes:**
- ❌ `rf!label` — wrong prefix
- ❌ Using database column name (`LABEL`) instead of field name (`label`)
- ❌ Missing spaces in record type name ("CMStatus" vs "CM Status")

## Sequential Relationship Creation

Add relationships to the same record type **one at a time**. Each relationship addition increments the record type's `versionId`. Parallel additions to the same record type cause 409 version conflict errors.

```
Add relationship "status" (versionId=2)    → succeeds, versionId now 3
Add relationship "priority" (versionId=3)  → succeeds, versionId now 4
```

If you get a 409, fetch the current record type to get the latest `versionId` and retry.

## Sample Data

Every database-backed record type needs sample data after creation.

### Quantity Guidelines

- **Reference tables**: All specified values (3-10 complete vocabulary)
- **Primary entities**: 15-20 diverse, realistic examples
- **Junction tables**: 20-30 relationships showing varied patterns

### Distribution Patterns

For primary entities, provide variety:
- **Across categories**: 3-4 per status/priority (not all in one)
- **Across time**: Span 3-6 months (not all same day)
- **Across users**: 2-4 per assignee (not concentrated)
- **Value ranges**: Realistic spreads ($50-$5000, not all $100)

### Reference Table Sample Data

List specific values:
```
STATUS: Created, Assigned, Approved, Closed
PRIORITY: Low, Medium, High, Critical
```

Each gets: id (auto), label (specified), sortOrder (1,2,3...), isActive (true)

### Primary Entity Sample Data

Provide domain-appropriate examples:
```
MAINTENANCE_REQUEST: Generate 18 examples with realistic titles
(HVAC System Failure, Electrical Outlet Repair, Water Leak, etc.)
Distribute: 4-5 per priority, 3-4 per status, span last 4 months,
assign 3-5 requests per technician
```

### Junction Table Sample Data

Show relationship variety:
```
STUDENT_COURSE: Generate 25 enrollments showing students in multiple
courses. Some students in 2-3 courses, vary enrollment dates across
2-3 semesters
```

### USER Field Values (MANDATORY)

Before generating sample data with USER fields, query `SYSTEM_RECORD_TYPE_USER` to get real usernames. Never fabricate usernames — they must match actual platform users for relationship navigation to work.
