---
name: "appian-change-review"
description: "Verify changes after making them to Appian applications — re-read created or modified objects, confirm cross-references are established correctly, detect discrepancies between intended and actual state, and interpret error responses. Use when the agent has just created or modified Appian design objects and needs to confirm the changes landed correctly."
---

## Post-Change Verification

After creating or modifying any Appian design object, re-read it to confirm the change applied correctly. Don't assume success from a non-error response — verify the actual state matches your intent.

### What to Verify by Object Type

**Record types:**
- After creating a record type, retrieve it and check that the name, plural name, and description match what you specified
- Get the source configuration to confirm all fields were created with the correct field types, primary key designation, and length constraints
- If you set `createTable: true`, confirm the source configuration reflects the database table name, data source UUID, and schema

**Record type relationships:**
- After adding a relationship, list the record type's relationships to confirm it appears
- Verify the relationship name, type (ONE_TO_MANY, MANY_TO_ONE, ONE_TO_ONE), and target record type UUID are correct
- Check the other side — retrieve the target record type's relationships to confirm the bidirectional relationship was added

**Interfaces:**
- After creating an interface, retrieve it to confirm the name and description
- After adding inputs, list the interface's inputs to confirm each one has the correct name, type QName, and multiplicity
- After setting or updating the SAIL expression, get the expression back and verify it matches what you sent

**Expression rules:**
- After creating an expression rule, retrieve it to confirm the name, description, inputs, and expression body
- Verify that input names and type QNames match your intent
- Confirm the expression body starts with `=` and references the correct `ri!` input names

**Process models:**
- After creating a process model, retrieve it to confirm nodes, variables, and connections
- Verify the start form configuration — check that `interfaceInformation` references the correct interface UUID and that `ruleInputs` map correctly
- Confirm the security group name matches an existing group

**Sites:**
- After creating or updating a site, retrieve it to confirm all pages are present in the correct order
- Verify each page's `targetUuid` points to the correct interface
- Check that visibility expressions are set on the pages that need them

**Web APIs:**
- After creating a Web API, retrieve it to confirm the name, URL alias, HTTP method, and request body type
- If you set an expression body, retrieve it and verify it matches

**Constants, groups, folders, documents:**
- After creating any supporting object, retrieve it to confirm the name, type, and value (for constants) or parent relationship (for folders and groups)

### When to Skip Verification

- Deleting an object — if the delete call succeeds, the object is gone; no need to re-read
- Listing objects — read-only operations don't change state; nothing to verify
- Simple metadata updates (renaming an object) — low risk; verify only if the rename is critical to downstream references

## Cross-Reference Verification

Many Appian objects reference each other. After establishing cross-references, verify both sides to catch partial or broken links.

### Record Type Relationships

Relationships must be declared on both sides for Appian record type traversal to work:

1. After adding a MANY_TO_ONE relationship on the FK side, list relationships on the target record type to confirm the corresponding ONE_TO_MANY exists
2. After adding a ONE_TO_MANY relationship, list relationships on the child record type to confirm the corresponding MANY_TO_ONE exists
3. For many-to-many via junction tables, verify all four relationship declarations: two on the junction table (MANY_TO_ONE to each entity) and one ONE_TO_MANY on each entity pointing to the junction table

### Interface Inputs and SAIL Expressions

The SAIL expression references inputs via `ri!` — a mismatch causes runtime errors:

1. After setting a SAIL expression, list the interface's inputs
2. For each `ri!` reference in the expression, confirm a matching input exists with the correct name and type
3. If the expression references an input that doesn't exist, add the missing input before the interface is used

### Process Model Start Forms

Start forms connect process models to interfaces with mapped inputs:

1. After creating a process model with `interfaceInformation`, verify the interface UUID exists by retrieving the interface
2. Check that each `ruleInput` name in the start form configuration matches an actual input on the interface
3. Verify that `value` references (e.g., `pv!record`) correspond to process variables that exist on the process model

### Process Model Node References

Nodes may reference interfaces, record types, or expression rules:

1. For User Input Task nodes, verify the `interfaceUuid` in the configuration points to an existing interface
2. For Write Records nodes, verify the `recordTypeUuid` points to an existing record type
3. For Script Task nodes that reference expression rules via `rule!`, confirm those expression rules exist

### Site Page References

Site pages point to interfaces:

1. After creating or updating a site, verify each page's `targetUuid` by retrieving the referenced interface
2. If a page has a `visibilityExpr` that references constants (e.g., `cons!PREFIX_ADMIN_GROUP`), verify those constants exist and have the correct type (GROUP)

### Application Inventory

After creating objects with `appUuid`, verify they appear in the application:

1. List objects of the relevant type scoped to the application's `appUuid`
2. Confirm the newly created object appears in the list
3. If it doesn't appear, add it to the application explicitly using the tool that associates objects with an application

## Discrepancy Handling

When verification reveals a mismatch between what you intended and what actually exists, diagnose the issue and attempt a corrective action.

### Common Discrepancies and Corrections

**Missing field on a record type:**
- Symptom: a field you specified in `sourceAndCustomFields` doesn't appear when you list the record type's fields
- Likely cause: the field definition had an invalid `fieldType` or was malformed
- Correction: get the current source configuration, add the missing field with a valid field type, and update the record type with the complete field list

**One-sided relationship:**
- Symptom: a relationship exists on one record type but the corresponding relationship is missing on the other side
- Likely cause: you added the relationship on one side but forgot the other, or the second add failed
- Correction: add the missing relationship on the other record type — MANY_TO_ONE on the FK side needs a corresponding ONE_TO_MANY on the target, and vice versa

**Interface input missing:**
- Symptom: the SAIL expression references `ri!inputName` but the input doesn't exist on the interface
- Likely cause: the input was not added before the expression was set, or the add-input call failed
- Correction: add the missing input with the correct name and type QName, then verify the expression still works

**SAIL expression not set:**
- Symptom: retrieving the interface expression returns empty or a default placeholder
- Likely cause: the expression update call failed silently or the expression had a syntax error
- Correction: re-send the expression, ensuring it starts with `=` and all `ri!` references match existing inputs

**Process model start form mismatch:**
- Symptom: the process model's `interfaceInformation` references an interface input name that doesn't exist on the interface
- Likely cause: the interface input was renamed or the mapping used the wrong name
- Correction: either update the process model's `ruleInputs` to use the correct input name, or add the missing input to the interface

**Object not in application inventory:**
- Symptom: listing objects scoped to the application doesn't include a recently created object
- Likely cause: the `appUuid` was not passed during creation, or the association failed
- Correction: use the tool that adds objects to an application, providing the object's UUID and type identifier

**Site page pointing to non-existent interface:**
- Symptom: retrieving the interface UUID from a page's `targetUuid` returns a not-found error
- Likely cause: the interface was deleted, or the wrong UUID was used
- Correction: create the missing interface or update the site page with the correct interface UUID

### Discrepancy Reporting

When you detect a discrepancy:

1. State what you expected vs. what you found
2. Explain the likely cause
3. Describe the corrective action you're taking
4. After applying the correction, re-verify to confirm the fix worked
5. If the correction fails, report the issue to the developer with enough detail for them to investigate

## Error Interpretation

Errors from the Appian platform fall into categories that determine whether to retry, adapt your approach, or escalate to the developer.

### Retry (Transient Errors)

These errors are temporary and may succeed on a second attempt:

- **Timeout or connection errors** — the platform was temporarily unavailable; wait briefly and retry once
- **500 Internal Server Error with no specific message** — may be a transient platform issue; retry once
- **Rate limiting (429)** — you're making requests too quickly; wait and retry with a brief pause

Retry at most once. If the same error occurs on retry, treat it as a persistent error and adapt or escalate.

### Adapt (Wrong Parameters or Approach)

These errors mean your request was malformed or used incorrect values. Don't retry with the same parameters — fix the issue first:

- **400 Bad Request** — the request body is malformed or missing required fields. Check the error message for specifics. Common causes: missing required field, invalid field type name, wrong QName format, duplicate name.
- **404 Not Found** — the object UUID you referenced doesn't exist. The object may have been deleted, or you're using the wrong UUID. Re-discover the object by listing and searching.
- **409 Conflict** — a uniqueness constraint was violated. Common causes: duplicate record type name, duplicate Web API method + URL alias combination, duplicate constant name. Change the conflicting value and retry.
- **422 Unprocessable Entity** — the request is syntactically valid but semantically wrong. Common causes: invalid relationship configuration, circular dependency, field type mismatch.

When adapting:

1. Read the error message carefully — it usually identifies the specific problem
2. Fix the identified issue in your request
3. Re-send the corrected request
4. Verify the result after the corrected request succeeds

### Escalate (Permissions or Missing Prerequisites)

These errors require developer intervention — the agent cannot resolve them:

- **401 Unauthorized** — authentication failed. The agent's credentials are invalid or expired. Report to the developer.
- **403 Forbidden** — the agent doesn't have permission to perform this action. The object or operation requires higher privileges. Report to the developer with the specific action that was denied.
- **Missing data source** — if creating a database-backed record type fails because no data source is available, the developer needs to configure a data source in the Appian environment.
- **Missing connected system** — if a process model node references an integration or connected system that doesn't exist, the developer needs to create it manually (this is an untooled capability).
- **Platform configuration issues** — errors related to environment configuration (database connectivity, external system availability, license limits) require developer or admin intervention.

### Error Response Patterns

When an error occurs:

1. **Read the full error message** — don't just check the status code; the message body often contains the specific field or value that caused the problem
2. **Check your inputs against the error** — compare what you sent with what the error says is wrong
3. **Don't retry blindly** — if the error is a 400 or 422, retrying with the same parameters will produce the same error
4. **Log what happened** — when reporting to the developer, include: what you were trying to do, the error status code and message, and what you've already tried

### Recovery After Errors

If an error occurs partway through a multi-step operation (e.g., creating a record type succeeded but adding relationships failed):

1. **Assess what completed** — re-read the objects that were part of the operation to determine current state
2. **Don't start over** — if the record type was created successfully, don't delete it and recreate; continue from where the error occurred
3. **Fix forward** — address the specific failure and continue with the remaining steps
4. **Re-verify the full result** — after recovering, verify the entire set of changes, not just the step that failed

## Runtime Verification

After creating or updating any interface, run `testInterface` to catch runtime errors that syntax validation misses.

### Interface Checks

After every `createInterface` or `updateInterface`, call `testInterface` and verify:

1. `diagnostics.error` must be null — any value means a runtime rendering failure
2. No `"-1"` values in rendered text — indicates a failed `.totalCount` or broken query
3. Grid column data must contain strings or proper component objects — NOT serialized `"[@attributes=..."` text (indicates a link/complex object placed directly in a value slot)
4. Filtered grids must return fewer rows than total record count — proves the filter parameter is working
5. Alert/count cards must show non-negative integers

If any check fails: identify the root cause from the rendered component tree, fix the expression, re-test until all checks pass, then proceed.

### Site Checks

After creating a site, GET it back and verify:
- Each page has a valid `targetUuid` (retrieve the interface to confirm it exists)
- Each `visibilityExpr` is a valid expression (e.g., `"true"`, `"=true"`, or a group membership check)

### Process Model Checks

After creating nodes, list all nodes and verify:
- The connection graph forms a valid path from Start (node 1) to End (node 2) — every node must be reachable
- XOR gateways have `decision.conditions` with valid `targetNodeId` values that reference existing nodes
- No orphan nodes (nodes with no incoming or outgoing connections, except Start and End)

### Cross-Object Wiring

After building multiple objects that reference each other (e.g., process model + interface + record action), explicitly verify the connections:

1. Record actions: retrieve the record type's actions and confirm `processModelUuid` points to an existing process model
2. Start forms: retrieve the process model and confirm `startForm.interfaceUuid` points to an existing interface
3. Summary views: retrieve the record type's views and confirm `interfaceExpression` references an existing interface via `rule!`
4. Site pages: retrieve the site and confirm each page's `targetUuid` points to an existing interface

Do not assume cross-references are correct because individual object creation succeeded — verify the wiring explicitly.
