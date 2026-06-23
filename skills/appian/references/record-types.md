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

**This relationship is MANDATORY and should be added automatically** (Proactive Completion pattern). When you create or add a USER field, immediately add the required relationship without asking the user. See "Confirmation and Completion Patterns" section below for the complete workflow.

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

## Confirmation and Completion Patterns

This section covers record type-specific patterns for when to ask users for confirmation versus when to automatically complete mandatory steps. Always load `references/confirmation-patterns.md` for universal workflows.

**Relationship between this section and confirmation-patterns.md:**
- **Universal file** provides the conceptual workflow (what to do)
- **This section** provides the executable details (exactly how to do it for record types)

Load both files together - universal for structure, this file for specifics.

### Delete Operations

**When to use:** Before deleting record types, fields, relationships, views, actions, or user filters.

**Cross-reference:** `confirmation-patterns.md` → Universal Workflow 1 (Delete Confirmation) for overall workflow structure. This section provides the specific dependency checks for record types.

#### Record Type Deletion

Record type deletion is **CRITICAL risk** and requires typed confirmation. Follow these dependency checks:

1. **Check relationships** via `listRecordTypeRelationships(uuid)`
   - Count MANY_TO_ONE relationships (this RT depends on others)
   - Count ONE_TO_MANY relationships (other RTs depend on this one)
   - Each relationship that references this RT will break

2. **Check views** via `listRecordTypeViews(uuid)`
   - Each view will be deleted along with the record type
   - Views may be referenced in sites or interfaces

3. **Check actions** via `listRecordTypeActions(uuid)`
   - Each action will be deleted
   - Actions may be referenced in interfaces or process models

4. **Check data** via `listRecordData(uuid, limit=1)`
   - If any records exist, data will become inaccessible
   - Database table is PRESERVED (see confirmation-patterns.md for details)

**Present to user:**
```
You are about to DELETE record type "[Name]" (UUID: [uuid]).

This will permanently remove:
- Appian record type definition (metadata layer)
- [N] relationships to other record types
- [N] record views
- [N] record actions
- [N] user filters

Database impact:
- The database table [TABLE_NAME] will be PRESERVED
- [N] data records remain in database but are no longer accessible via Appian

Impact on other objects:
- [List of record types with FK relationships to this one] will have orphaned fields
- Interfaces/expressions using this record type will break

This action CANNOT be undone (record type metadata will be lost).

To confirm, type: DELETE [Name]
```

#### Field Deletion

Field deletion is **HIGH risk** and requires Yes/No confirmation if the field is in use.

**Dependency checks:**

1. **Check relationships** via `listRecordTypeRelationships(uuid)`
   - Is this field used as `sourceRecordTypeFieldUuid`? (FK field in MANY_TO_ONE)
   - Is this field used as `targetRecordTypeFieldUuid`? (PK field in relationship)
   - If yes, relationship must be deleted first

2. **Check title expression** via `getRecordType(uuid)`
   - Does `titleExpression` reference this field's UUID?
   - If yes, title expression will break

3. **Check views** via `listRecordTypeViews(uuid)`
   - Review each view's displayed fields
   - If field is displayed, view will show empty column

4. **Check security expressions** (cannot detect automatically)
   - Warn user: "Manual verification needed if field used in security expressions"

**Present to user if field is in use:**
```
You are about to DELETE field "[fieldName]" from record type "[RecordTypeName]".

This field is currently used in:
- [N] relationships (list relationship names)
- Title expression
- [N] record views

Deleting this field will break these references. Continue? (yes/no)
```

**If field not in use, proceed with lower-risk confirmation:**
```
Delete field "[fieldName]"? This field is not currently used in relationships, views, or title expression. Continue? (yes/no)
```

#### Relationship Deletion

Relationship deletion is **HIGH risk** if it's the only navigation path between record types.

**Check before deleting:**
- Is there a reverse relationship? (MANY_TO_ONE ↔ ONE_TO_MANY pair)
- If deleting one side, should both sides be deleted?

**Present to user:**
```
You are about to DELETE relationship "[relationshipName]" ([Type]: [SourceRT] → [TargetRT]).

[If reverse exists] The reverse relationship "[reverseName]" will remain. Users can still navigate [TargetRT] → [SourceRT] but not [SourceRT] → [TargetRT].

[If no reverse] This is the only navigation path between these record types.

Continue? (yes/no)
```

**Recommendation to AI:** For bidirectional relationships, suggest deleting both sides to avoid one-way navigation confusion.

### Name Collision Detection

**When to use:** Before creating record types or fields.

**Cross-reference:** `confirmation-patterns.md` → Universal Workflow 2 (Name Collision Detection)

#### Record Type Name Collision

Record types are **application-scoped**. Check within the current application only.

**Mandatory Step 1:** Call `listRecordTypes(appUuid)` BEFORE `createRecordType`

**Similarity rules:**
- Same app + exact match → COLLISION (ask user)
- Same app + same prefix + related purpose → POTENTIAL COLLISION (ask user)
  - Example: "PM Status" vs "PM Priority" (both lookups)
  - Example: "PM Case" vs "PM CaseStatus" (subset name)
- Different app → NO COLLISION (proceed without asking)

**Present to user if collision detected:**
```
Found existing record type "[ExistingName]" in this application.

Would you like to:
1. Use the existing record type
2. Create a new record type with a different name
3. Proceed anyway (creates duplicate)

What would you like to do? (1/2/3)
```

#### Field Name Collision

Fields are scoped to their parent record type. Check within the same record type only.

**Mandatory Step 1:** Check `fields` array from `getRecordType(uuid)` or creation response

**Present to user if exact match found:**
```
Record type "[RTName]" already has a field named "[fieldName]".

Cannot create duplicate field name. Please choose a different name.
```

### Proactive Completion Patterns

**Key principle:** These are **mandatory requirements**, not user choices. Complete them automatically and inform the user what was done.

#### USER Field Relationships (MANDATORY)

**Rule:** Every USER field MUST have a MANY_TO_ONE relationship to `SYSTEM_RECORD_TYPE_USER`.

**When to trigger:** Immediately after creating a record type with USER fields, or after adding a USER field to an existing record type.

**Workflow:**
1. Detect USER field in creation response (field with `fieldType: "USER"`)
2. Automatically call `addRecordTypeRelationship`:
   - `relationshipType`: "MANY_TO_ONE"
   - `sourceRecordTypeUuid`: the record type UUID
   - `sourceRecordTypeFieldUuid`: the USER field UUID
   - `targetRecordTypeUuid`: `SYSTEM_RECORD_TYPE_USER`
   - `targetRecordTypeFieldUuid`: `SYSTEM_RECORD_TYPE_USER_FIELD_username`
   - `relationshipName`: `{fieldName}User` (e.g., `assignedTo` → `assignedToUser`)
3. Inform user: "✅ Created USER field '[fieldName]' with required relationship to system User record type"

**Do NOT ask:** "Should I add a relationship to the User record type?" — this is mandatory, not optional.

**Example from validated test:**
```
Created record type "PMS Task" with fields:
- assignedTo (USER)
- createdBy (USER)
- modifiedBy (USER)

✅ Automatically added 3 required relationships:
- Task.assignedToUser → User (MANY_TO_ONE)
- Task.createdByUser → User (MANY_TO_ONE)
- Task.modifiedByUser → User (MANY_TO_ONE)
```

**Cross-reference:** `references/relationship-patterns.md` for detailed USER field relationship pattern.

#### Bidirectional Relationships (MANDATORY)

**Rule:** FK relationships require both MANY_TO_ONE (FK table → referenced table) AND ONE_TO_MANY (referenced table → FK table) for bidirectional navigation.

**When to trigger:** After adding a MANY_TO_ONE relationship.

**Workflow:**
1. User requests or AI adds MANY_TO_ONE relationship (e.g., Task.project → Project)
2. Automatically add reverse ONE_TO_MANY relationship:
   - On the **target** record type (Project)
   - Pointing back to the **source** record type (Task)
   - Use pluralized relationship name (e.g., `tasks`)
3. Ask for missing info only: "What should the reverse relationship name be on [TargetRT]? (suggest: '[pluralizedName]')"
4. Add both relationships sequentially
5. Inform user: "✅ Added bidirectional relationships: [SourceRT].[fkName] and [TargetRT].[reverseName]"

**Do NOT ask:** "Should I add the reverse relationship?" — this is mandatory per Appian best practices.

**Example from validated test:**
```
User: "Add relationship from Task to Project"

AI: Checks if reverse relationship exists
AI: "Task.projectId → Project.id (MANY_TO_ONE).
     Will also add reverse Project → Task (ONE_TO_MANY) for bidirectional navigation.
     What should the relationship name be on Project? (suggest: 'tasks')"
     
User: "tasks"

AI: [Adds Task.project MANY_TO_ONE, then Project.tasks ONE_TO_MANY]
    "✅ Added bidirectional relationships: Task.project and Project.tasks"
```

**Cross-reference:** `references/relationship-patterns.md` for complete bidirectional pattern details.

#### Sample Data Offering

**When to use:** After successfully creating a database-backed record type.

**Pattern type:** INFORMATIVE — neither confirmation nor proactive completion.

**Workflow:**

After creating a record type, inform user that no sample data was added:

```
✅ Created [RecordTypeName] record type with [N] fields.

Note: No sample data added. To add test records, you can say:
- "Add sample data" (uses intelligent defaults: 15-20 records with realistic distribution)
- "Add 15 test records" (specify exact quantity)
- "Add realistic data" (applies full distribution patterns across categories/time/users)
```

**If user requests sample data:**

1. Follow decision rules below (see Sample Data Decision Rules section)
2. Generate realistic data based on field types:
   - TEXT fields: Realistic names/descriptions
   - INTEGER fields: Sequential IDs, reasonable numbers
   - USER fields: Query SYSTEM_RECORD_TYPE_USER (MANDATORY)
   - DATETIME fields: Recent dates with realistic spacing
   - FK fields: Reference existing records (query target record type)
3. Insert via `insertRecordData` (CSV format)
4. Inform: "✅ Added [N] sample records"

**Why inform but not ask:**
- Sample data is optional, not mandatory
- User may want specific data, not generic samples
- Doesn't block workflow with unnecessary question

**Example:**

```
User: "Create Employee record type with fields: id, name, email, departmentId"

AI: [Creates record type]
    "✅ Created Employee record type with 4 fields (id, name, email, departmentId).
    
    Note: No sample data added. To add test records, you can say:
    - 'Add sample data' (uses intelligent defaults: 15-20 records with realistic distribution)
    - 'Add 15 test records' (specify exact quantity)
    - 'Add realistic data' (applies full distribution patterns across categories/time/users)"

User: "Add 15 sample employees"

AI: [Generates 15 realistic employees, queries department FK target]
    "✅ Added 15 sample employee records."
```

**Cross-reference:** For decision rules on quantity, distribution, and related record population, see Sample Data Decision Rules section below.

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

---

### Sample Data Decision Rules

When user requests sample data, follow these decision rules to determine when to ask for input vs when to use intelligent defaults.

#### 1. Quantity Decision

**User specifies count explicitly:**
- "Add 10 sample employees" → Use 10 (don't ask)
- "Add sample data with 20 rows" → Use 20 (don't ask)

**User says "add sample data" (no count):**
- **Reference tables (Status, Priority, Category):** Use ALL values from existing guidance (don't ask)
  - Status tables: Use exactly the statuses from Quantity Guidelines (Created, Assigned, Approved, Closed)
  - Priority tables: Use exactly Low, Medium, High, Critical (don't ask)
- **Primary entities (Case, Employee, Order):** Use 15-20 (existing guidance, don't ask)
- **Junction tables (many-to-many):** Use 20-30 (existing guidance, don't ask)

**Never ask:** "How many sample records?" unless user request is ambiguous.

**Rationale:** Existing guidance already defines appropriate quantities. Don't ask questions when guidance provides the answer.

#### 2. Distribution Decision

**User specifies distribution:**
- "Add cases with various statuses" → Use distribution patterns from existing guidance
- "Add 5 high priority cases" → All same priority (as specified)

**User says "add sample data" or "add realistic data":**
- Apply distribution patterns automatically (don't ask):
  - Across categories: 3-4 per status/priority (not all in one)
  - Across time: Span 3-6 months (not all same day)
  - Across users: 2-4 per assignee (not concentrated)
  - Value ranges: Realistic spreads ($50-$5000, not all $100)

**Never ask:** "What distribution do you want?" unless user specifies non-standard requirement.

**Rationale:** "Realistic data" and "sample data" mean apply existing distribution patterns. User requests patterns implicitly.

#### 3. Related Record Population

**Core principle:** Only populate the table user requested. Don't automatically populate related tables without asking.

**Foreign key field exists, target table is EMPTY:**
- ⚠️ BLOCKING ISSUE — Cannot proceed without target values
- Present issue and ask for resolution:
  ```
  Cannot add sample [SourceRecordType] records: the [TargetRecordType] table 
  (referenced by [fieldName]) is empty.
  
  To proceed, we need to populate [TargetRecordType] first. Options:
  1. You provide custom values (recommended if you know them)
  2. Use standard values (e.g., for Status: Created, Assigned, Approved, Closed)
  3. Skip sample data for now
  
  What would you like to do? (1/2/3)
  ```
- **Option 1 (recommended):** User provides custom values, AI creates target records, then creates source records
- **Option 2:** AI uses standard values from guidance, creates target records, then creates source records
- **Option 3:** Abort sample data generation
- **Why ask:** User may have specific domain values (e.g., "Draft, Review, Published" for content status)

**Foreign key field exists, target table HAS DATA:**
- Use existing values (don't ask, don't create more)
- Query target table: `listRecordData(targetRecordTypeUuid, limit=20)`
- Randomly distribute FK values across existing target records
- Only populate the table user requested (not related tables)
- Example: Status table has 4 statuses → distribute 15 cases across those 4 statuses

**Relationship is MANY_TO_ONE:**
- Only populate source table (the one with FK field)
- Don't add records to target table unless it's empty (see blocking issue above)
- Example: User says "add sample cases" → populate Case table only
  - If Status table has data: use existing statuses
  - If Status table empty: ask to populate Status first (blocking)

**Relationship is ONE_TO_MANY (inverse exists):**
- Only populate the table user requested
- Don't automatically populate the "many" side
- Example: User says "add sample projects" → populate Project table only
  - Don't auto-create Tasks even if Project.tasks relationship exists
  - User must explicitly say "add sample tasks" to populate Task table

**Rationale:** 
- Empty FK target = blocking issue (must resolve to proceed)
- Populated FK target = reuse existing values (maintains referential integrity)
- Don't auto-populate related tables (user asks for what they want)

#### 4. USER Field Values (MANDATORY - Always Query)

Before generating sample data with USER fields:
1. Query SYSTEM_RECORD_TYPE_USER: `listRecordData(uuid="SYSTEM_RECORD_TYPE_USER", limit=20)`
2. Extract real usernames from results
3. Randomly distribute across sample records
4. Never fabricate usernames (breaks relationship navigation)

**This is MANDATORY — not a decision point.**

**Rationale:** USER fields must reference real platform users. Fabricated values break navigation and cause errors.

#### 5. Examples by Request Type

| User Request | Quantity | Distribution | Related Records (FK Behavior) |
|---|---|---|---|
| "Add sample data" | Use defaults (15-20 for entities) | Apply patterns (vary status/time/users) | FK target has data → use existing; FK target empty → **ASK** to populate first |
| "Add 10 test cases" | Use 10 (specified) | Minimal (may not vary all dimensions) | FK target has data → use existing; FK target empty → **ASK** to populate first |
| "Add realistic cases" | Use defaults (15-20) | Full patterns (vary everything) | FK target has data → use existing; FK target empty → **ASK** to populate first |
| "Add high priority cases" | Use defaults (15-20) | All same priority (as specified) | FK target has data → use existing; FK target empty → **ASK** to populate first |
| "Add sample projects" (has Tasks relationship) | Use defaults (15-20) | Apply patterns | Don't auto-create Tasks (user didn't ask for them) |

#### 6. When to Ask vs When to Decide

**Always ask (blocking decisions):**
- FK target table is empty (cannot proceed without target records)
- User request is genuinely ambiguous (no guidance to infer from)
- **User says "add sample data" immediately after creating multiple record types (unclear which table to populate)**

**Never ask (use existing guidance):**
- Quantity for reference/primary/junction tables (guidance defines defaults)
- Distribution patterns for "realistic" or "sample" data (guidance defines patterns)
- Whether to query USER table (mandatory, not optional)
- Whether to reuse existing FK values (standard practice, don't create duplicates)

**Key Principle:** Don't ask questions when existing guidance provides the answer. Apply intelligent defaults unless there's a genuine blocking issue or ambiguity.

---

### Ambiguous Sample Data Requests

**When ambiguity occurs:**

User creates multiple record types in same conversation, then says "add sample data" without specifying which table.

**Example scenarios:**

| Context | Ambiguity | Why Ambiguous |
|---|---|---|
| User creates Employee and Department tables, then says "add sample data" | Which table? One? Both? | Two tables created, unclear target |
| User creates Product and Category tables, then says "add sample data" | Which table? In what order? | FK relationship exists, order matters |
| User creates single Vendor table, then says "add sample data" | No ambiguity | Only one table, clear target |

**How to clarify:**

Present options clearly:
```
You just created [N] record types: [Name1], [Name2][, ...].

Which would you like to add sample data to?
1. [Name1] ([brief description or type, e.g., "reference table"])
2. [Name2] ([brief description or type, e.g., "entity table"])
3. Both ([Name1] first, then [Name2])

What would you like to do? (1/2/3)
```

**After user clarifies:**
- **Option 1 or 2:** Proceed to populate specified table, check for FK blocking issues if applicable
- **Option 3:** Populate in dependency order (reference tables before entities)

**Example:**

```
User: Creates "Order Status" and "Order" tables
User: "Add sample data"

AI: "You just created 2 record types: Order Status and Order.

     Which would you like to add sample data to?
     1. Order Status (reference table - status values)
     2. Order (entity table - orders)
     3. Both (Order Status first, then Order)
     
     What would you like to do? (1/2/3)"

User: "2"

AI: [Queries Order Status table]
    [Detects empty] → Triggers blocking issue with 3 options (custom/standard/skip)
```

**Cross-reference:** For handling ambiguous requests in general, see `confirmation-patterns.md` → Universal Workflow 4 (Ambiguous Request Clarification).
