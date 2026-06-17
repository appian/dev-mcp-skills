---
name: "appian-record-types"
description: "Create and modify Appian record types including fields, relationships, and source configuration. Covers field types, naming conventions, relationship management, sample data generation, and database table creation. Use when working with record types, data models, database tables, or record type relationships."
---

## Orchestration: Check Before Creating

Before creating a record type, discover what exists to avoid duplication.

1. **List record types**: Use list tool with `appUuid` to see all in application
2. **Search for related entities**: Look for entities that might relate
   - Creating "Order"? Check for "Customer", "Product", "OrderStatus"
   - Creating "CaseNote"? Check for "Case"
3. **Ask the user**: Present findings and get confirmation
   - "Found 'Customer' record type. Should 'Order' reference it?"
   - "No 'Priority' reference table. Create new or use TEXT field?"

**Pattern: Discover → Ask → Decide**

Always: (1) List what exists, (2) Present options, (3) Follow user's decision

## Relevant MCP Tools

- **createRecordType** — name, source configuration (fields, table settings), optional plural name, description
- **listRecordTypes** / **getRecordType** — retrieve by UUID or search with filtering; scope with `appUuid`
- **updateRecordType** — modify name, description, plural name, source configuration, relationships
- **deleteRecordType** — permanently remove by UUID
- **listRecordTypeFields** / **getRecordTypeField** — retrieve fields or specific field by name
- **addRecordTypeRelationship** / **listRecordTypeRelationships** — add/retrieve relationships
- **getRecordTypeSourceConfiguration** — retrieve data source, table name, schema, fields

## Creating Record Types

### Source Configuration

For database-backed record types:
- `sourceType`: "DATABASE"
- `createTable`: true (Appian creates table)
- `tableName`: UPPER_SNAKE_CASE with application prefix (e.g., `JTA_CASE`)
- `dataSourceUuid` and `schema`: Discover from existing record types if available
- `sourceAndCustomFields`: Define all fields

### Field Configuration

Each field requires:
- `fieldName`: camelCase record type field name (e.g., `statusId`, `createdBy`, `updatedAt`)
- `fieldType`: One of 12 valid types (see field-types reference)
- `isPrimaryKey`: Exactly one field must be true; use `id` (INTEGER)
- Optional: `displayName`, `description`, `isUnique`, `length` (for TEXT)
- Optional: `sourceFieldName`: Database column name (defaults to UPPER_SNAKE_CASE of fieldName)

**Important:** Distinguish between record type field names (camelCase) and database column names (UPPER_SNAKE_CASE):
- Record field `statusId` → Database column `STATUS_ID`
- Record field `createdBy` → Database column `CREATED_BY`
- Record field `updatedAt` → Database column `UPDATED_AT`

### Naming Conventions (WITH Application Prefix)

**Record Type Names:**
- Format: "[Prefix] Entity Name" in Title Case
- Examples: "JTA Case", "JTA Status", "JTA Priority"
- Plural: "JTA Cases", "JTA Statuses", "JTA Priorities"

**Database Table Names:**
- Format: `[PREFIX]_[ENTITY]` in UPPER_SNAKE_CASE
- Examples: `JTA_CASE`, `JTA_STATUS`, `JTA_PRIORITY`
- Always singular: `JTA_CASE` not `JTA_CASES`

**Record Type Field Names (camelCase):**
- Examples: `statusId`, `createdBy`, `updatedAt`, `isActive`
- Primary key: `id`
- Foreign keys: `[referencedEntity]Id`
- Boolean: `is[Condition]`
- Dates: `[context]Date` or `[context]At`

**Database Column Names (UPPER_SNAKE_CASE):**
- Examples: `STATUS_ID`, `CREATED_BY`, `UPDATED_AT`, `IS_ACTIVE`
- Primary key: `ID`
- Foreign keys: `[REFERENCED_TABLE]_ID`
- Appian auto-converts from camelCase fieldName if sourceFieldName not specified

### Field Ordering

Add fields in this sequence (matches UI display order):
1. Primary key: `id` (INTEGER)
2. Descriptive fields: name, title, description
3. Classification fields: statusId, priorityId, categoryId (FKs)
4. Temporal fields: requestDate, dueDate, completionDate
5. Quantitative fields: amount, quantity, cost
6. User reference fields: requestedBy, assignedTo, approvedBy (USER type)
7. Audit fields: createdAt, createdBy, updatedAt, modifiedBy (USER type)
8. Soft delete fields (if applicable): isDeleted, deletedAt, deletedBy

### Field Descriptions

When planning phase provides field descriptions in implementation notes:
- Format: `fieldName (TYPE, required/optional) - description text`
- Strip markers before passing to tool: [PROPOSED], [DERIVED], (Source: Requirements)
- Only include descriptions documented in planning; don't fabricate

**Example:**
```
Planning notes: isActive (BOOLEAN) - Controls visibility in dropdowns [PROPOSED]
Tool call: description="Controls visibility in dropdowns"
```

### Unique Constraints

Apply single-field uniqueness using `isUnique` parameter when planning documents:
- "UNIQUE constraint: email" → `isUnique=True`
- "UNIQUE constraint: employeeNumber" → `isUnique=True`

Compound uniqueness (e.g., "UNIQUE constraint: (orderId, lineNumber)") cannot be enforced at database level — document only for application logic.

## Managing Relationships

### Operation Sequence

1. Create all record types with fields first
2. After record types exist, add relationships
3. Get field UUIDs from creation responses (never guess)

**Important:** Add relationships **sequentially** to avoid version conflicts:
- Each `addRecordTypeRelationship` increments the record type's `versionId`
- Adding relationships in parallel to the same record type causes 409 errors
- Solution: Add one relationship at a time, OR fetch current `versionId` before each batch

### Bidirectional Relationships (MANDATORY)

Every FK relationship requires TWO relationships:

**One-to-Many:**
```
CASE has statusId (FK → STATUS.id)

Required relationships (both directions):
1. CASE.statusId → STATUS.id (MANY_TO_ONE) — "Get status for case"
2. STATUS.id → CASE.statusId (ONE_TO_MANY) — "Get all cases with status"
```

**Junction Tables (Four Relationships):**
```
STUDENT_COURSE has studentId (FK) and courseId (FK)

Required relationships (four total):
1. STUDENT_COURSE.studentId → STUDENT.id (MANY_TO_ONE)
2. STUDENT.id → STUDENT_COURSE.studentId (ONE_TO_MANY)
3. STUDENT_COURSE.courseId → COURSE.id (MANY_TO_ONE)
4. COURSE.id → STUDENT_COURSE.courseId (ONE_TO_MANY)
```

### Adding Relationships

Each relationship requires:
- `relationshipName`: camelCase (see naming algorithm below)
- `sourceRecordTypeFieldUuid`: FK field UUID on source
- `targetRecordTypeFieldUuid`: Referenced field UUID (usually PK) on target
- `targetRecordTypeUuid`: UUID of target record type
- `relationshipType`: ONE_TO_MANY, MANY_TO_ONE, or ONE_TO_ONE

### Relationship Naming

Derive algorithmically from camelCase field names:
- **MANY_TO_ONE**: Remove `Id` suffix (`statusId` → `status`)
- **ONE_TO_MANY to entities**: Pluralize entity in camelCase (`CASE_NOTE` → `caseNotes`)
- **ONE_TO_MANY to junction**: Pluralize other entity (`CASE → CASE_TAG → TAG` = `tags`)
- **Self-referential inverse**: Semantic plural (`parentCaseId` inverse → `childCases`)

### USER Field Relationships

Every USER field must have MANY_TO_ONE relationship to SYSTEM_RECORD_TYPE_USER:
- System UUID: `SYSTEM_RECORD_TYPE_USER`
- User PK UUID: `SYSTEM_RECORD_TYPE_USER_FIELD_username`
- Relationship name: `{fieldName}User` (e.g., `createdBy` → `createdByUser`)
- Note: Use `createdBy`, `modifiedBy` (NOT `createdByUsername`, `modifiedByUsername`)

## Sample Data (MANDATORY)

Every database-backed record type needs sample data immediately after creation.

### Quantity Guidelines

**Default volume (supports dashboards):**
- Reference tables: All specified values (3-10 complete vocabulary)
- Primary entities: 15-20 diverse, realistic examples
- Junction tables: 20-30 relationships showing varied patterns

**User adjustments:**
- "Minimal data" / "quick prototype" → 3-5 entities, 5-8 junction
- "Rich data" / "comprehensive" → 40-50 entities, 60-80 junction

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

**USER field values:**
Before generating sample data with USER fields, query `SYSTEM_RECORD_TYPE_USER` to get real usernames:
```
listRecordData(uuid="SYSTEM_RECORD_TYPE_USER", limit=20)
```
Use actual usernames from the response for `createdBy`, `modifiedBy`, `assignedTo`, etc. Never fabricate usernames.

### Junction Table Sample Data

Show relationship variety:
```
STUDENT_COURSE: Generate 25 enrollments showing students in multiple 
courses. Some students in 2-3 courses, vary enrollment dates across 
2-3 semesters
```

## Record Title Expression (REQUIRED)

Set title expression after creation using `updateRecordType`:
- Reference tables: Use `label` field
- Primary entities: Use most identifying field (title, name, number)
- Format: `rv!record['recordType!{recordTypeUuid}RecordTypeName.fields.{fieldUuid}fieldName']`

**Example:**
```
rv!record['recordType!{259f1bf5-a915-427b-9db3-d49ebefee9cf}JTA Status.fields.{6580097e-8856-4704-b42b-3753ad76ad3d}label']
```

**Construction:**
1. Get record type UUID and name from creation response
2. Get field UUID for the title field (label, name, title, etc.)
3. Use exact record type name including spaces
4. Use exact field name (camelCase)

Title expression determines text in record links, views, headers.

## Record Type Descriptions

Include clear 1-2 sentence description:
- Appears in Appian Designer for developers
- Guides execution agent for sample data
- Serves as documentation

Strip planning markers before passing to tool: [PROPOSED], [DERIVED], (Source: Requirements)

**Example:**
```
Planning: "Tracks equipment maintenance requests [PROPOSED]"
Tool call: description="Tracks equipment maintenance requests"
```

## User Filters

When requirements mention filtering, add User Filters directly to record type (not interface-level):

**Map to facet types:**
- FK with M:1 relationship → LIST_OF_VALUES (useRelatedRecordValues)
- Date/Datetime → DATE_RANGE
- Text fields → EXPRESSION (a!recordFilterList with aggregation)
- Boolean → LIST_OF_VALUES (explicit Yes/No)
- Dynamic/computed → EXPRESSION

Add AFTER creating record type but BEFORE inserting sample data.

## Updating Record Types

### Read Before Update

Always retrieve current state first. Update operation replaces provided fields entirely.

1. Get record type to see current configuration
2. Get source configuration to see current fields
3. Merge changes with existing state
4. Send complete updated configuration

### Adding Fields

Retrieve source configuration, append new fields to `sourceAndCustomFields`, update with complete list.

## Troubleshooting

### Version Conflict (HTTP 409)

**Error:** "The provided versionId is invalid or outdated"

**Cause:** Record type was modified between when you got the versionId and when you tried to update it.

**Solution:**
1. Fetch current record type with `getRecordType` to get latest `versionId`
2. Retry the operation with updated `versionId`
3. For multiple relationships, add them sequentially instead of in parallel

**Example:**
```
# ❌ This causes 409 errors (parallel modifications)
addRelationship(uuid, "status", versionId="2")      # succeeds, versionId now 3
addRelationship(uuid, "priority", versionId="2")    # fails 409 - versionId is stale

# ✅ Better: sequential
addRelationship(uuid, "status", versionId="2")      # succeeds, versionId now 3
addRelationship(uuid, "priority", versionId="3")    # succeeds, versionId now 4
```

### Title Expression Errors (HTTP 400)

**Error:** "Unresolved reference(s): rf!label" or similar

**Cause:** Incorrect title expression syntax.

**Correct Format:**
```
rv!record['recordType!{uuid}RecordTypeName.fields.{fieldUuid}fieldName']
```

**Common mistakes:**
- ❌ `rf!label` — incorrect prefix
- ❌ `fv!row.label` — wrong context
- ❌ Missing quotes around the full path
- ❌ Using database column name (LABEL) instead of field name (label)
- ❌ Missing spaces in record type name ("JTAStatus" vs "JTA Status")

**Debug steps:**
1. Verify record type UUID from creation/get response
2. Verify field UUID from `fields` array
3. Use exact record type name (including spaces)
4. Use camelCase field name, not database column name

## Common Pitfalls

- **Not checking existing record types** — always discover first to avoid duplicates
- **Forgetting bidirectional relationships** — both sides required (MANY_TO_ONE + ONE_TO_MANY)
- **One-sided relationships** — breaks record type traversal
- **Composite primary keys** — Appian requires single `id` field
- **Updating without reading first** — loses existing fields
- **Missing `createTable: true`** — expects table to already exist
- **Wrong field type names** — case-sensitive; use exact values
- **Forgetting `length` on TEXT** — defaults to 0 (unlimited)
- **Audit fields on reference/junction tables** — only on primary entities
- **Creating user entities** — use USER field type instead
- **Omitting application prefix** — table and record type names need prefix
- **Wrong PK field name** — must be `id`, not `case_id` or `customer_id`
- **Missing sample data** — record type incomplete without it
- **No title expression** — required for all record types
- **Wrong title expression format** — use full `rv!record['recordType!{uuid}...']` syntax, not `rf!field`
- **Forgetting USER field relationships** — must link to SYSTEM_RECORD_TYPE_USER
- **Fabricating usernames in sample data** — query SYSTEM_RECORD_TYPE_USER for real usernames
- **Version conflicts on relationships** — add sequentially, not in parallel

## Design Decisions: When to Use Data Modeling Skill

Before creating record types, **activate `appian-data-modeling` skill** for:
- Entity classification (5-criteria test for reference vs entity vs junction)
- Normalization decisions (when to normalize vs denormalize)
- Relationship design (which type, when to use junction)
- Field structures (patterns for entities, reference tables, junctions)
- Naming conventions (tables, fields, relationships)

Then use this skill to implement with MCP tools.

**Boundary:** `appian-data-modeling` = WHAT and WHY (design). This skill = HOW (implementation).

## Related Skills

**Activate `appian-data-modeling` skill** for design guidance on:
- Schema design principles (normalization, 3NF, 4NF)
- Entity vs Reference vs Junction classification
- Relationship design patterns
- Naming conventions and validation rules
- When to normalize vs denormalize

This skill focuses on record type lifecycle and MCP tool mechanics.

## When You Need More

For comprehensive patterns, load reference files:
- `references/composer/planning.md` — Platform-specific rules (238 lines)
- `references/composer/execution.md` — Tool mechanics and sample data (591 lines)

For questions beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation.

Load `references/field-types.md` for complete field type reference with constraints.
