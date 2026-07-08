# Appian MCP Tool Reference

## Table of Contents

- [Discovery](#discovery)
- [Key Behaviors](#key-behaviors)
  - Updates are partial replacement
  - SAIL expressions are strings
  - CSV format for record data
  - UUIDs are always in responses
- [UUID Handling](#uuid-handling)
  - Critical Rule: Never Fabricate UUIDs
  - UUID Sources
  - Storing UUIDs for Multi-Step Operations
  - Sequential Operations Pattern
  - UUID Retrieval Decision Tree
  - Common UUID Mistakes
  - Verification
- [Multi-Step Patterns](#multi-step-patterns)
  - Record type creation sequence
  - Connected system configuration loop
  - Process model node type discovery
  - Integration configuration loop
  - Complete data model creation
- [Non-Obvious Behaviors](#non-obvious-behaviors)
  - Process model requirements
  - Record type relationships
  - Record actions
  - Record events
  - testInterface and testRule
  - validateDesignObject vs validateExpression
- [Error Patterns](#error-patterns)

---

## Discovery

Appian platform operations are available as MCP tools. The tools are self-describing — inspect their parameter schemas directly for field names, types, and descriptions.

Tool names follow a pattern like `mcp__appian__<toolName>` (the exact prefix depends on server configuration). Look for characteristic tool names like `createApplication`, `createRecordType`, `listInterfaces`, `getProcessModel` regardless of prefix.

This reference covers patterns, conventions, and non-obvious behaviors that tool schemas don't communicate well.

## Key Behaviors

### Updates are partial replacement
Most update tools accept only the fields you want to change — omitted fields are preserved. However, **array fields are full replacement**: if you provide `nodes` on `updateProcessModel`, you must include ALL nodes (not just new ones). Same for `pages` on `updateSite`, `inputs` on `updateInterface`, and `processVariables` on `updateProcessModel`.

### SAIL expressions are strings
Pass SAIL expressions as plain string values. No special escaping beyond normal JSON string escaping. The `expression` field on interfaces and expression rules must start with `=`.

### CSV format for record data
`insertRecordData`, `updateRecordData`, and `deleteRecordData` all use CSV format (header row + data rows), not JSON arrays. Boolean values must be `1`/`0` (not `true`/`false`). Dates use `YYYY-MM-DD`, datetimes use `YYYY-MM-DD HH:MM:SS`.

### UUIDs are always in responses
Every create tool returns the new object's `uuid` directly in the response. No special output formatting needed — just read it from the JSON response.

### Before calling create operations

**MANDATORY pre-create workflow** — follow these steps BEFORE calling any `create` tool:

1. **List existing objects** (environmental awareness)
   - Creating an application → call `listApplications` first
   - Creating a record type → call `listRecordTypes(appUuid)` first
   - Creating an expression rule → call `listExpressionRules(appUuid)` first
   - Creating an interface → call `listInterfaces(appUuid)` first
   - Creating a group → call `listGroups` first
   - Creating a constant → call `listConstants(appUuid)` first
   - Creating a process model → call `listProcessModels(appUuid)` first

2. **Check for name collisions** per `confirmation-patterns.md` → Universal Workflow 2
   - Compare proposed name against existing objects
   - Same-app + similar name → ask user before creating
   - Different-app prefix → proceed without asking

3. **Only then call the create operation**

**Why this matters:**
- Prevents duplicate objects with similar names
- Detects potential conflicts before creating
- Provides user with informed choice when collisions exist

**See also:** `references/confirmation-patterns.md` for complete name collision detection workflow and examples.

## UUID Handling

### Critical Rule: Never Fabricate UUIDs

**NEVER invent, guess, or fabricate UUIDs:**
- ❌ Guessing UUID format (`_field-abc123`, `_recordtype-xyz789`)
- ❌ Reusing UUIDs from examples or documentation
- ❌ Fabricating placeholder UUIDs ("I'll use this temporary UUID...")
- ❌ Copying UUIDs from different environments (dev UUID in prod)
- ❌ Assuming UUID patterns are predictable
- ❌ Using UUIDs from previous conversations without verification

**Why this matters:**
- Invalid UUIDs cause silent failures (404 Not Found)
- Wrong UUIDs can modify/delete the wrong objects
- Placeholder UUIDs break subsequent operations
- Cross-environment UUIDs cause data corruption

**If you don't have a UUID: STOP and retrieve it from the environment using list/get tools.**

### UUID Sources

UUIDs come from exactly three sources:

#### 1. Creation Responses (Primary Source)

When creating an object, the response contains its UUID and any nested object UUIDs (fields, parameters, etc.):

```json
{
  "uuid": "_recordtype-abc123",
  "name": "JTA Case",
  "fields": [
    {"uuid": "_field-def456", "fieldName": "id"},
    {"uuid": "_field-ghi789", "fieldName": "statusId"}
  ]
}
```

Extract UUIDs immediately from the response for use in subsequent operations.

#### 2. Environment Retrieval (When UUIDs Not Available)

When UUIDs aren't available from creation or prior steps, retrieve them from the environment:

**Common retrieval patterns:**
- Record type UUID → `listRecordTypes` (filter by name)
- Field UUIDs → `listRecordTypeFields` (filter by name)
- Interface UUID → `listInterfaces` (filter by name)
- Constant UUID → `listConstants` (filter by name)
- Group UUID → `listGroups` (filter by name)
- Process model UUID → `listProcessModels` (filter by name)

Always filter by name to find the specific object, then extract the UUID from the response.

#### 3. System Constants (Platform UUIDs)

Some UUIDs are platform constants that work across all environments:
- `SYSTEM_RECORD_TYPE_USER` — system user record type
- `SYSTEM_RECORD_TYPE_USER_FIELD_username` — username field in system user record type
- `SYSTEM_RECORD_TYPE_GROUP` — system group record type

These are **literal string values**, not variables to retrieve. Use exactly as shown.

### Storing UUIDs for Multi-Step Operations

For workflows that create multiple objects with dependencies, store UUIDs explicitly after each creation:

**Pattern: Include UUIDs in response text**
```markdown
✅ Created Case entity

UUIDs for subsequent steps:
- Record Type: _recordtype-case555
- id field [PK]: _field-aaa111
- statusId field [FK]: _field-bbb222
- priorityId field [FK]: _field-ccc333
- title field [TITLE]: _field-ddd444
```

This makes UUIDs visible to the user and available for lookup in conversation history.

### Sequential Operations Pattern

Many operations must be sequential because later steps need UUIDs from earlier steps:

**Example: Creating record type with relationships**
```
1. createRecordType → get record type UUID and field UUIDs
2. Store UUIDs explicitly in response
3. addRecordTypeRelationship (use UUIDs from step 1)
4. addRecordTypeRelationship (use updated versionId from step 3)
5. insertRecordData (use field UUIDs from step 1)
```

Each step depends on UUIDs from previous steps. Never proceed to a dependent step without the required UUIDs.

### UUID Retrieval Decision Tree

```
Need UUID for an operation?
├─ Was object just created in this workflow?
│  └─ YES → Use UUID from creation response ✅
│  └─ NO → Continue
│
├─ UUID available from earlier in conversation?
│  └─ YES → Extract from prior response ✅
│  └─ NO → Continue
│
└─ UUID not available?
   └─ MUST retrieve from environment using list/get tools ✅
   └─ NEVER guess or fabricate ❌
```

### Common UUID Mistakes

- **Fabricating UUIDs** — Creating placeholder values instead of retrieving real ones
- **Reusing example UUIDs** — Documentation examples are not real UUIDs
- **Cross-environment UUIDs** — Dev UUIDs don't work in prod (each environment has separate UUIDs)
- **Stale UUIDs** — Using UUIDs from previous conversations without verification

**See also:** `references/confirmation-patterns.md` (Universal Workflow 3: UUID Verification) for when to verify UUIDs before operations.
- **Wrong object UUIDs** — Using field UUID where record type UUID is required, or vice versa
- **Missing field UUIDs** — Forgetting that `sourceRecordTypeFieldUuid` and `targetRecordTypeFieldUuid` must be actual field UUIDs, not fabricated

### Verification

When retrieving UUIDs from the environment, verify you found the correct object:
- Check the `name` field matches what you're looking for
- Verify the object type is correct (record type vs field vs interface)
- Confirm the parent/container matches (e.g., field belongs to the correct record type)

## Multi-Step Patterns

### Record type creation sequence
A complete record type setup spans multiple calls:
1. `createRecordType` (with fields) → get UUID and field UUIDs from response
2. `addRecordTypeRelationship` (after ALL related record types exist)
3. `addRecordTypeView` (needs interface UUIDs)
4. `addRecordTypeAction` (needs process model UUIDs)
5. `addRecordTypeUserFilter` (needs field UUIDs)

### Connected system configuration loop
Connected systems use an iterative pattern:
1. `createConnectedSystem` → response includes `schema` (available properties) and `properties` (current values, mostly null)
2. Read the schema to see what fields are available
3. `updateConnectedSystem` with the fields you want to set
4. If you changed a discriminator field (e.g., `authType`), the schema changes — repeat from step 2

### Process model node type discovery
Before configuring unfamiliar node types:
1. `getProcessModelNodeTypeSchema` with the type ID (e.g., `"internal3.write_records_to_source_23r3"`)
2. Use `referenceUuid` parameter to enrich schema with context from a referenced object (integration inputs, subprocess PVs)
3. Use `formInterfaceUuid` for attended nodes to discover form input mappings

### Integration configuration loop
Same iterative pattern as connected systems:
1. `createIntegration` → response includes `schema` and `properties`
2. Read schema, set properties via `updateIntegration`
3. Repeat — schema may change based on operation or discriminator values
4. Properties marked `isExpressionable=true` accept SAIL expressions; for literal strings, wrap in quotes: `'"my value"'`

### Complete data model creation

When creating a data model with multiple record types, relationships, and sample data, follow this order to avoid dependency errors and ensure completeness:

**Phase 1: Reference Tables**
1. `createRecordType` for each reference/lookup table (Status, Priority, Category, etc.)
   - Store record type UUID and field UUIDs from each response
2. `insertRecordData` for each reference table
   - Insert all lookup values (all statuses, all priorities)
3. `updateRecordType` to set title expression on each reference table
   - Use `rv!record['recordType!{uuid}Name.fields.{fieldUuid}fieldName']` format
   - Reference tables typically use `label` or `name` field

**Phase 2: Entity Tables**
4. `createRecordType` for main entity tables (with FK fields to references and USER fields)
   - Include INTEGER fields for FKs (e.g., `statusId`, `priorityId`)
   - Include USER fields for assignments (e.g., `assignedTo`, `createdBy`)
   - Store record type UUID and ALL field UUIDs from response

**Phase 3: Relationships (Sequential, Not Parallel)**
5. `addRecordTypeRelationship` for each FK relationship (MANY_TO_ONE from entity → reference)
   - Use stored UUIDs from creation responses
   - Each call increments `versionId` — must be sequential on same record type
   - Name relationships by removing `Id` suffix: `statusId` → `status`
6. `addRecordTypeRelationship` for reverse relationships (ONE_TO_MANY from reference → entity)
   - Add on the reference record type, pointing back to entity
   - Use pluralized name: `Status.tickets`, `Priority.tickets`
7. `addRecordTypeRelationship` for USER field relationships (MANY_TO_ONE to SYSTEM_RECORD_TYPE_USER)
   - **Required for every USER field** — see `references/relationship-patterns.md`
   - Use platform constants: `SYSTEM_RECORD_TYPE_USER`, `SYSTEM_RECORD_TYPE_USER_FIELD_username`
   - Name pattern: `{fieldName}User` (e.g., `assignedTo` → `assignedToUser`)
   - Sequential with other relationships on same record type

**Phase 4: Sample Data & Title Expressions**
8. Query real usernames before inserting entity data
   - `listRecordData` on `SYSTEM_RECORD_TYPE_USER` to get valid usernames
   - Never fabricate usernames — they must match actual platform users
9. `insertRecordData` for entity tables
   - Use real FK values from reference table IDs
   - Use queried usernames for USER fields
   - Provide realistic distribution (vary statuses, priorities, assignments)
10. `updateRecordType` to set title expression on entity tables
    - Typically use `title` or `name` field for entities

**Why this order matters:**
- FKs require target record types to exist (Phase 1 before Phase 2)
- Relationships require both record types to exist (Phase 2 before Phase 3)
- Sample data requires relationships to exist for navigation to work (Phase 3 before Phase 4)
- Title expressions should be set after data exists for testing
- Sequential relationship operations avoid 409 version conflicts

**Common mistakes:**
- Adding relationships before all record types exist → 404 errors
- Adding relationships in parallel to same record type → 409 version conflicts
- Skipping USER field relationships → navigation fails, displays plain text
- Skipping reverse (ONE_TO_MANY) relationships → one-way navigation only
- Fabricating usernames in sample data → relationship navigation fails
- Setting title expressions before relationships exist → can't test navigation

**Example sequence for Help Desk (Status, Priority, Ticket):**
```
1. createRecordType(Status)           → store Status UUID + field UUIDs
2. insertRecordData(Status, 4 rows)
3. updateRecordType(Status, titleExpression for label field)
4. createRecordType(Priority)         → store Priority UUID + field UUIDs
5. insertRecordData(Priority, 4 rows)
6. updateRecordType(Priority, titleExpression for label field)
7. createRecordType(Ticket with statusId, priorityId, assignedTo) → store all UUIDs
8. addRecordTypeRelationship(Ticket → Status via statusId)    [versionId: 2→3]
9. addRecordTypeRelationship(Ticket → Priority via priorityId) [versionId: 3→4]
10. addRecordTypeRelationship(Ticket → User via assignedTo)    [versionId: 4→5]
11. addRecordTypeRelationship(Status → Ticket, ONE_TO_MANY)
12. addRecordTypeRelationship(Priority → Ticket, ONE_TO_MANY)
13. listRecordData(SYSTEM_RECORD_TYPE_USER) → get real usernames
14. insertRecordData(Ticket, 15-20 rows with real statusId/priorityId/usernames)
15. updateRecordType(Ticket, titleExpression for title field)
```

This 15-step sequence ensures a complete, working data model with full bidirectional navigation.

## Non-Obvious Behaviors

### Process model requirements
- `parentFolderUuid` must be a **process model folder** (from `listProcessModelFolders`), not a regular folder
- `errorAlertGroupName` is required on create — use the app's administrators group name
- Start forms use `inputMap` where keys = PV names (without `pv!`), values = interface input names (without `ri!`)

### Record type relationships
- Must declare both sides: MANY_TO_ONE on the FK table AND ONE_TO_MANY on the referenced table
- Both record types must exist before either relationship can be added
- `sourceRecordTypeFieldUuid` and `targetRecordTypeFieldUuid` must be real field UUIDs from create/get responses — never fabricate them

### Querying with relationships

When creating a query that includes related fields, **you must retrieve field UUIDs from BOTH the base record type AND all related record types**:

**Discovery workflow:**
1. `getRecordType(baseRecordTypeUuid)` → get base fields and relationship UUIDs
2. For each related record type referenced in the query:
   - `getRecordType(relatedRecordTypeUuid)` → get related record type's field UUIDs
3. Build query using UUIDs from both

**Why this matters:**
- Base record type response includes relationship UUIDs but NOT the field UUIDs from related record types
- You cannot guess or fabricate field UUIDs for related fields
- Without proper UUIDs, queries fail with "field was invalid" errors

**Example:**
```
Goal: Query Case with related Status fields

1. getRecordType(caseUuid) → discover:
   - Case field UUIDs: id, title, statusId
   - Relationship UUID: status → links to Status record type

2. getRecordType(statusUuid) → discover:
   - Status field UUIDs: id, label, sortOrder

3. Build query:
   'recordType!{caseUuid}.fields.{idUuid}id',
   'recordType!{caseUuid}.relationships.{statusRelUuid}status.fields.{statusIdUuid}id',
   'recordType!{caseUuid}.relationships.{statusRelUuid}status.fields.{labelUuid}label'
```

**Common mistake:**
```
❌ getRecordType(caseUuid) only → guess Status field UUIDs → query fails
✅ getRecordType(caseUuid) + getRecordType(statusUuid) → use real UUIDs → query succeeds
```

### Record actions
- `contextExpr` keys must match process model parameter names (case-sensitive)
- `icon` is a Font Awesome hex code (e.g., `"f044"`), not a name (not `"pencil"`)
- RELATED_ACTIONs only surface on record views, not on custom `a!gridField` grids

### Record events
- `configureRecordEvents` returns 409 if already configured — check with `getRecordEventsConfig` first
- Event types are managed after setup by inserting/updating records in the Event Type Lookup record type (UUID from config response)
- Events can only be written via Write Records node in process models — not via `a!writeRecords()`

### testInterface and testRule
- `testInterface` renders the interface and returns the component tree + any `diagnostics.error` entries
- Use it to catch runtime errors that expression validation misses (bad record references, type mismatches)
- `testRule` (type: `EXPRESSION_RULE` or `INTEGRATION`) executes with provided inputs and returns the result

### validateDesignObject vs validateExpression
- `validateDesignObject` checks all expressions on a saved object (by UUID)
- `validateExpression` validates a raw SAIL expression without saving — no `ri!` or record references available unless you provide bindings

## Error Patterns

- **Expression evaluation errors** on properties marked `isExpressionable` — you likely need to quote a literal string value: `'"my value"'` not `'my value'`
- **409 Conflict** — object already exists or is already configured (record events)
- **Validation errors after update** — check that renamed/removed inputs don't break `ri!` references in expressions
- **Missing parent** — folders need `parentFolderUuid`, process models need PM folder UUID, documents need document folder UUID

**See also:** `references/confirmation-patterns.md` for interactive workflows when deleting objects, detecting name collisions, or completing mandatory steps.

## Handling Tool Limitations

Not every operation can be completed with the available tools. When you hit one that can't, stop retrying and move on.

The tools do not cover every Appian object type, configuration option, or field type. When an error indicates the operation itself isn't supported — as opposed to a correctable input mistake — flag it as a manual step for the user rather than continuing to retry.

When surfacing a manual step:
- State the action clearly (e.g., "Create a constant of type Process Model in Appian Designer")
- Provide context values generated during automation (UUIDs, names) so the user can complete it without re-discovering them
- Continue with the rest of the plan — don't block on one unsupported operation unless downstream steps depend on its output
