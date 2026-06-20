# Appian MCP Tool Reference

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

## Non-Obvious Behaviors

### Process model requirements
- `parentFolderUuid` must be a **process model folder** (from `listProcessModelFolders`), not a regular folder
- `errorAlertGroupName` is required on create — use the app's administrators group name
- Start forms use `inputMap` where keys = PV names (without `pv!`), values = interface input names (without `ri!`)

### Record type relationships
- Must declare both sides: MANY_TO_ONE on the FK table AND ONE_TO_MANY on the referenced table
- Both record types must exist before either relationship can be added
- `sourceRecordTypeFieldUuid` and `targetRecordTypeFieldUuid` must be real field UUIDs from create/get responses — never fabricate them

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

## Handling Tool Limitations

Not every operation can be completed with the available tools. When you hit one that can't, stop retrying and move on.

The tools do not cover every Appian object type, configuration option, or field type. When an error indicates the operation itself isn't supported — as opposed to a correctable input mistake — flag it as a manual step for the user rather than continuing to retry.

When surfacing a manual step:
- State the action clearly (e.g., "Create a constant of type Process Model in Appian Designer")
- Provide context values generated during automation (UUIDs, names) so the user can complete it without re-discovering them
- Continue with the rest of the plan — don't block on one unsupported operation unless downstream steps depend on its output
