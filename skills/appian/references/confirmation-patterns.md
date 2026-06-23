# Confirmation Patterns Reference

## When to Load This File

Load this file BEFORE creating, updating, or deleting any Appian objects. It defines:
- When to ask the user for confirmation (Interactive Confirmation)
- When to automatically complete mandatory steps (Proactive Completion)
- Universal workflows that apply across all object types

**Load alongside `references/tools-mcp.md` in Step 1 of every Appian task.**

---

## Table of Contents

**Pattern Definitions**
- [Pattern Type Definitions](#pattern-type-definitions)
- [Decision Tree](#decision-tree)

**Universal Workflows**
- [Universal Workflow 1: Delete Confirmation](#universal-workflow-1-delete-confirmation)
- [Universal Workflow 2: Name Collision Detection](#universal-workflow-2-name-collision-detection)
- [Universal Workflow 3: UUID Verification](#universal-workflow-3-uuid-verification)
- [Universal Workflow 4: Ambiguous Request Clarification](#universal-workflow-4-ambiguous-request-clarification)
- [Universal Workflow 5: Proactive Completion Patterns](#universal-workflow-5-proactive-completion-patterns)
- [Universal Workflow 6: Rename/Update Confirmation](#universal-workflow-6-rename-update-confirmation)
- [Universal Workflow 7: Dependency Checking](#universal-workflow-7-dependency-checking)

**Special Case Templates**
- [Record Type Delete Special Case](#record-type-delete-special-case)
- [Constant Delete Special Case](#constant-delete-special-case)
- [Expression Rule Breaking Changes Special Case](#expression-rule-breaking-changes-special-case)

**Proactive Completion Patterns**
- [Proactive Completion: USER Field Relationships](#proactive-completion-user-field-relationships)
- [Proactive Completion: Bidirectional Relationships](#proactive-completion-bidirectional-relationships)

---

<a name="pattern-type-definitions"></a>
## Pattern Type Definitions

### Interactive Confirmation

**Definition:** Patterns where the user must make a choice or accept a risk before the AI proceeds.

**When to use:**
- Destructive operations (deletes, overwrites)
- Ambiguous requests (missing required information)
- Name collisions or conflicts with existing objects
- UUID verification before high-risk operations

**Key principle:** The user is making a **decision** — the AI presents options and waits for user choice.

### Proactive Completion

**Definition:** Patterns where the AI must automatically complete mandatory steps without asking.

**When to use:**
- Mandatory relationships required by Appian platform (USER field → SYSTEM_RECORD_TYPE_USER)
- Bidirectional relationship pairs (FK field → MANY_TO_ONE + ONE_TO_MANY)
- Platform requirements documented as "MUST" or "MANDATORY" in skill references

**Key principle:** These are **requirements**, not choices — the AI completes them automatically and informs the user what was done.

### Decision Tree

```
Is this a platform/skill MANDATORY requirement?
├─ YES → Proactive Completion (auto-complete, inform user)
└─ NO → Does this involve risk or choice?
    ├─ YES → Interactive Confirmation (ask user first)
    └─ NO → Proceed normally
```

---

<a name="universal-workflow-1-delete-confirmation"></a>
## Universal Workflow 1: Delete Confirmation

### When to Use

Apply this workflow before ANY destructive operation:
- Deleting applications, record types, fields, relationships, views, actions
- Deleting expression rules, interfaces, constants, process models
- Deleting groups, folders, documents, sites
- Removing relationships or dependencies

### Risk Levels

| Risk Level | Scope | Examples | Confirmation Type |
|---|---|---|---|
| **CRITICAL** | Affects multiple objects or production data | Application delete, Record Type delete, Process Model delete, **Constant delete**, **Expression Rule breaking changes** (input removal, type changes) | Typed confirmation or 3-option choice |
| **HIGH** | Affects single object with potential dependencies | Field delete, Relationship delete, Expression Rule delete (no callers) | Yes/No confirmation |
| **MEDIUM** | Affects single object, limited dependencies | View delete, Action delete | Yes/No confirmation |

**Notes:** 
- Constants were reclassified from MEDIUM to CRITICAL based on Phase 2.5 validation testing. Reason: Constants are name-based (`cons!NAME`) not UUID-based. Deleting breaks ALL expressions using that constant with no automatic updates.
- Expression Rule breaking changes (input removal, input type change, return type change) are CRITICAL because they break ALL callers with no automatic updates. Expression Rule deletion without breaking changes remains HIGH.

### Workflow Steps

1. **Receive delete request** from user
   - User may provide name, UUID, or description
   - May be explicit ("delete X") or implicit ("remove that")

2. **Verify object existence**
   - Call appropriate `get` or `list` operation to resolve identifier
   - If UUID provided, verify it's valid
   - If name provided, resolve to UUID
   - If ambiguous, ask for clarification

3. **Extract object details**
   - Name, type, creation date, last modified
   - Key metadata (number of fields, relationships, dependencies)
   - Application context if applicable

4. **Check dependencies** (object-specific)
   
   **⚠️ IMPORTANT:** Use Universal Workflow 7 (Dependency Checking) to perform comprehensive dependency analysis.
   
   Universal Workflow 7 provides:
   - Structural dependency checks (relationships, views, actions, data)
   - Manual verification templates (for expression-based dependencies)
   - 3-option presentation format (Cancel recommended / Proceed / Alternative)
   - Clear communication of what will break
   
   **The confirmation message MUST include:**
   1. Structural dependencies found (with counts)
   2. Manual verification section (if expression-based checks not available)
   3. Options with Cancel recommended until verification complete
   
   See [Universal Workflow 7: Dependency Checking](#universal-workflow-7-dependency-checking) for:
   - Step-by-step dependency checking workflow
   - Presentation templates
   - Manual verification guidance templates
   - Examples for each object type

5. **Present to user with consequences**
   
   **Use the template from Universal Workflow 7 (Step 4: Present Dependencies to User).**
   
   The confirmation MUST follow this structure:
   ```
   ⚠️ You are about to DELETE [ObjectType] "[Name]"
   
   Structural dependencies (confirmed):
   ❌ [List all structural dependencies found]
   
   Manual verification required (cannot detect automatically):
   ⚠️ [List expression-based dependencies that need manual checks]
   
   [Use manual verification template from Workflow 7]
   
   Impact:
   - [List specific impacts]
   
   Options:
   1. Cancel (recommended until manual verification complete)
   2. Proceed with [OPERATION] (you are responsible for manual updates)
   3. [Alternative approach if available]
   
   What would you like to do? (1/2/3)
   ```
   
   For CRITICAL operations (Application, Record Type, Process Model, Constant delete):
   - Replace "What would you like to do? (1/2/3)" with
   - "Type 'DELETE [Name]' to confirm HIGH RISK operation."
   
   For CRITICAL breaking changes (Expression Rule input removal/type change):
   - Keep "What would you like to do? (1/2/3)" format
   - Present 3 alternatives (make optional / create v2 / cancel)
   
   **Use object-specific templates when available:**
   - Record Type delete → See [Record Type Delete Special Case](#record-type-delete-special-case)
   - Constant delete → See [Constant Delete Special Case](#constant-delete-special-case)
   - Expression Rule breaking changes → See [Expression Rule Breaking Changes Special Case](#expression-rule-breaking-changes-special-case)
   
   **Example for HIGH risk:**
   ```
   You are about to DELETE the "assignedTo" field from the Case record type.
   
   This field is currently used in:
   - 1 relationship (to User record type)
   - 2 record views
   - 1 record action security expression
   
   Deleting this field will break these references. Continue? (yes/no)
   ```

6. **Execute delete operation**
   - Only after user confirms with exact match (CRITICAL) or yes (HIGH/MEDIUM)
   - Call appropriate MCP delete tool
   - Inform user of success or failure

### Templates

**CRITICAL risk confirmation:**
```
You are about to DELETE [ObjectType] "[Name]" (UUID: [uuid]).

This will permanently delete:
[List of affected objects with counts]

This action CANNOT be undone.

To confirm, type: DELETE [Name]
```

**HIGH/MEDIUM risk confirmation:**
```
You are about to DELETE [ObjectType] "[Name]".

[Optional: Impact statement if dependencies found]

Continue? (yes/no)
```

<a name="record-type-delete-special-case"></a>
### Record Type Delete Special Case

**Important:** `deleteRecordType` removes the **Appian metadata** (record type definition, relationships, views, actions) but does NOT drop the **database table**.

**Use Universal Workflow 7 (Dependency Checking) first, then present confirmation using this template:**

```
⚠️ You are about to DELETE record type "[Name]" (UUID: [uuid])

This will permanently remove:
- Appian record type definition (metadata layer)
- [N] relationships to other record types
- [N] record views
- [N] record actions
- [N] user filters

Database impact:
- The database table [TABLE_NAME] will be PRESERVED
- [N] data records remain in the database but will no longer be accessible via Appian
- To drop the table, manual database administration is required

Structural dependencies (confirmed):
❌ [List relationships found via listRecordTypeRelationships]
❌ [List views found via listRecordTypeViews]
❌ [List actions found via listRecordTypeActions]
❌ [List data found via listRecordData]

Manual verification required (cannot detect automatically):
⚠️ Interfaces using 'recordType!{uuid}[Name]'
⚠️ Expression rules referencing this record type
⚠️ Process models writing to this record type

Manual verification REQUIRED:
1. Search expression rules for "recordType!{uuid}[Name]"
2. Search interfaces for "[Name]" record type references
3. Search process models for record type references in:
   - Start forms
   - Script tasks
   - Record actions

Use Appian Designer:
- Right-click record type → "Find Usages" (if available)
- Global search for "[Name]" and review results

What breaks:
❌ All expressions using recordType!{uuid}[Name] will fail
❌ All interfaces displaying this record type will error
❌ All process models writing to this record type will fail

Options:
1. Cancel (recommended until manual verification complete)
2. Proceed with delete (you are responsible for fixing breaks)

Type 'DELETE [Name]' to confirm HIGH RISK operation.
```

**Why this matters:**
- User expectations: clarifies that data is preserved
- Prevents confusion when database table still exists
- Explains two-layer architecture (Appian metadata vs database)

---

<a name="constant-delete-special-case"></a>
### Constant Delete Special Case

**Important:** Constants are referenced by **NAME** in expressions (`cons!CONSTANT_NAME`), not by UUID. Deleting a constant breaks **ALL** expressions that reference it, with **no automatic updates**.

**Use Universal Workflow 7 (Dependency Checking) first, then present confirmation using this template:**

```
⚠️ CRITICAL: You are about to DELETE constant "[Name]"

Constant details:
- Type: [TYPE] (GROUP, TEXT, INTEGER, etc.)
- Value: "[value]"
- Description: "[description]"
- UUID: [uuid]

⚠️ Cannot automatically detect all dependencies.

Manual verification REQUIRED before proceeding:
1. Search ALL expression rules for "cons![Name]"
2. Search ALL interfaces for "[Name]"
3. Search ALL process models for security expressions using this constant
4. Search ALL record actions for visibility/security expressions
5. Check Web API security expressions

Use Appian Designer:
- Open Constants page
- Right-click "[Name]" → "Find Usages" (if available)
- Review ALL usage locations
- Document every location before proceeding

What breaks if you proceed:
❌ ALL expression rules using cons![Name] will fail at runtime
❌ Security checks using this constant will break (may grant unauthorized access)
❌ Process assignments using this constant may fail
❌ Interface visibility conditions using this constant will error

What's preserved:
✅ The referenced object itself (e.g., the group "[value]" is not deleted)
✅ Other constants are not affected

Options:
1. Cancel (recommended until manual verification complete)
2. Proceed with delete (you verified no dependencies exist and are responsible for fixing any breaks)

Alternative approach (RECOMMENDED):
- Instead of deleting, update the constant's value to reference a different object
- This avoids breaking everything at once
- Allows gradual migration if needed

Type 'DELETE [Name]' to confirm HIGH RISK operation.
```

**Why this matters:**
- Constants are name-based references (unlike UUID-based record types)
- No automatic update mechanism when constant is deleted
- Expression failures can be silent (no compile-time checks)
- Security implications if constant controls access

---

<a name="expression-rule-breaking-changes-special-case"></a>
### Expression Rule Breaking Changes Special Case

**Important:** Expression rule breaking changes (input removal, input type change, return type change) break **ALL** calling rules, interfaces, and process models with **no automatic updates**.

**Use Universal Workflow 7 (Dependency Checking) first, then present confirmation using this template:**

```
⚠️ BREAKING CHANGE: Removing input "[inputName]" from [RuleName]

Current signature:
  [RuleName]([input1], [input2], [inputName], ...)

Proposed signature:
  [RuleName]([input1], [input2], ...)

Impact:
❌ ALL callers passing [inputName] will fail at runtime
❌ Expression rules calling this rule will break
❌ Interfaces using this rule will error
❌ Process models calling this rule will fail

⚠️ Cannot automatically detect all dependencies.

Manual verification REQUIRED before proceeding:
1. Search ALL expression rules for "rule![RuleName]"
2. Search ALL interfaces for "[RuleName]"
3. Search ALL process models for script task expressions calling this rule
4. Search ALL record actions/views for expressions using this rule
5. Check Web API expressions

Use Appian Designer:
- Open Expression Rules page
- Right-click "[RuleName]" → "Find Usages" (if available)
- Review ALL calling locations
- Document every caller before proceeding

What breaks if you proceed:
❌ All expression rules calling rule![RuleName]([inputName]: ...) will fail
❌ All interfaces calling this rule with [inputName] will error
❌ All process model script tasks using this rule will fail

Alternative approaches (choose one):
1. Make [inputName] optional with default value (NON-BREAKING):
   - Existing callers continue working
   - New callers can omit the parameter
   - Add default: a!defaultValue(ri![inputName], [defaultValue])
   
2. Create [RuleName]_v2 with new signature (NON-BREAKING):
   - Keep [RuleName] with old signature
   - Create new rule with different inputs
   - Migrate callers gradually
   - Delete old rule when migration complete
   
3. Cancel input removal (RECOMMENDED until verification complete)

What would you like to do? (1/2/3)
```

**For other breaking changes:**

**Input type change:**
```
⚠️ BREAKING CHANGE: Changing input "[inputName]" type from [OldType] to [NewType]

Current: [inputName] (type: [OldType])
Proposed: [inputName] (type: [NewType])

Impact:
❌ Existing callers passing [OldType] values will fail with type mismatch
❌ No automatic type conversion between [OldType] and [NewType]

[Same manual verification and options as above]
```

**Return type change:**
```
⚠️ BREAKING CHANGE: Changing return type from [OldType] to [NewType]

Current: Returns [OldType]
Proposed: Returns [NewType]

Impact:
❌ Callers expecting [OldType] will break when they receive [NewType]
❌ Interfaces displaying the value may error
❌ Process variables storing the result may have type mismatch

[Same manual verification and options as above]
```

**Why this matters:**
- Expression rules are called by NAME (rule!RuleName) not by UUID
- No compile-time checks (errors only appear at runtime)
- Breaking changes cascade through calling chain
- Hard to trace all dependencies without manual verification

---

### Cross-References

For object-specific dependency checks, see:
- Applications: `references/applications.md` (listApplicationObjects)
- Record Types: `references/record-types.md` (fields, relationships, views, actions)
- Process Models: `references/process-models.md` (nodes, start forms)

---

<a name="universal-workflow-2-name-collision-detection"></a>
## Universal Workflow 2: Name Collision Detection

### When to Use

Apply this workflow before creating ANY named object:
- Applications, record types, fields, relationships
- Expression rules, interfaces, constants
- Groups, process models, sites
- Any object with a `name` parameter

### Collision Scope Rules

**Application-scoped objects** (most objects):
- Check within the current application only
- Objects in different applications with same name are NOT collisions
- Applies to: record types, fields (within same record type), relationships, views, actions, constants, expression rules, interfaces, process models, sites

**Global objects** (shared across applications):
- Check across entire environment
- Applies to: groups, folders (depending on parent)

**Prefix-aware objects** (use application prefix pattern):
- Check for similar prefixes + similar names
- Applies to: constants, expression rules, interfaces, groups

### Similarity Detection Rules

When comparing names, apply these checks in order:

1. **Exact match** (case-sensitive)
   - "PM Status" = "PM Status" → COLLISION

2. **Case-insensitive match**
   - "PM Status" vs "pm status" → COLLISION

3. **Semantic similarity** (same prefix + related purpose)
   - "PM Status" vs "PM Priority" → POTENTIAL COLLISION (both lookup types)
   - "PM Case Status" vs "PM Status" → POTENTIAL COLLISION (subset relationship)
   - "PM Case" vs "PM Task" → POTENTIAL COLLISION (similar entity types)

4. **Different prefix** (different applications)
   - "PM Status" vs "CM Status" → NO COLLISION (proceed without asking)

### Workflow Steps

1. **MANDATORY - List existing objects BEFORE creating** (environmental awareness)

   Before any create operation, you MUST call the appropriate list operation:

   | Creating... | Call this FIRST | Scope |
   |---|---|---|
   | Application | `listApplications()` | Global |
   | Record Type | `listRecordTypes(appUuid)` | App-scoped |
   | Expression Rule | `listExpressionRules(appUuid)` | App-scoped |
   | Interface | `listInterfaces(appUuid)` | App-scoped |
   | Constant | `listConstants(appUuid)` | App-scoped |
   | Process Model | `listProcessModels(appUuid)` | App-scoped |
   | Group | `listGroups()` | Global |
   | Site | `listSites(appUuid)` | App-scoped |

   **Why this is mandatory:**
   - You cannot detect collisions without knowing what exists
   - "I'll create X" without checking → HIGH RISK of duplicates
   - List operations are fast and prevent costly mistakes

2. **Receive create request with name**
   - Extract proposed name from user request or parameters
   - Identify object type and scope (app-scoped or global)

3. **Compare proposed name against existing objects** (from Step 1 list results)
   - Extract names from list results
   - Identify which objects are in the same scope (same app, same prefix, etc.)

4. **Calculate similarity scores**
   - Check for exact match
   - Check for case-insensitive match
   - Check for semantic similarity (same prefix + related purpose)
   - Exclude different-prefix matches from consideration

5. **If similar names found, present matches**
   - Show existing object name, type, UUID
   - Show proposed name for comparison
   - Explain why they're similar (exact, case-insensitive, semantic)

6. **Ask user for decision**
   ```
   Found existing [ObjectType] with similar name in this application:
   - Existing: "[ExistingName]" (UUID: [uuid])
   - Proposed: "[ProposedName]"
   
   These are similar because: [same prefix + both lookup types / subset relationship / etc.]
   
   Options:
   1. Create new "[ProposedName]" (will be separate object)
   2. Use existing "[ExistingName]" instead
   3. Choose a different name
   
   What would you like to do? (1/2/3)
   ```

7. **Proceed based on choice**
   - Option 1: Proceed with create operation
   - Option 2: Return existing object UUID, skip create
   - Option 3: Ask user for new name, repeat workflow

### Examples by Object Type

**Record Types (app-scoped):**
```
User request: "Create a PM Status record type"

Check: listRecordTypes(appUuid: "pm-app-uuid")
Found: "PM Priority" (lookup type)

Similar because: Same prefix (PM) + both lookup types

Ask user: Create new "PM Status" or use existing "PM Priority"?
```

**Expression Rules (prefix-aware, app-scoped):**
```
User request: "Create PM_GetCaseStatus rule"

Check: listExpressionRules(appUuid: "pm-app-uuid")
Found: "PM_GetTaskStatus"

Similar because: Same prefix (PM_Get) + similar purpose (status retrieval)

Ask user: Create new "PM_GetCaseStatus" or use existing "PM_GetTaskStatus"?
```

**Groups (global, prefix-aware):**
```
User request: "Create CM Managers group"

Check: listGroups() [no app filter]
Found: "CM Administrators"

Similar because: Same prefix (CM) + both role groups

Ask user: Create new "CM Managers" or use existing "CM Administrators"?
```

**Different prefix (NO collision):**
```
User request: "Create PM Status record type"

Check: listRecordTypes(appUuid: "pm-app-uuid")
Found: "CM Status" in different application

Different prefix → NO COLLISION → Proceed without asking
```

### Semantic Similarity Categories

| Category | Examples | When to Flag |
|---|---|---|
| **Exact purpose** | Status/Status, Priority/Priority | Always |
| **Related lookups** | Status/Priority, Type/Category | Same prefix only |
| **Entity variants** | Case/Task, Customer/Client | Same prefix only |
| **Action variants** | Create/Add, Update/Edit, Delete/Remove | Same prefix + same context |
| **Subset names** | CaseStatus/Status, TaskPriority/Priority | Same prefix only |

### Cross-References

For object-specific scoping rules, see:
- Record Types: `references/record-types.md` (app-scoped)
- Expression Rules: `references/expression-rules.md` (app-scoped, prefix pattern)
- Groups: `references/supporting-objects.md` (global scope)

---

<a name="universal-workflow-3-uuid-verification"></a>
## Universal Workflow 3: UUID Verification

### When to Use

Apply this workflow before ANY operation that modifies or deletes an object by UUID:
- Update operations (updateRecordType, updateInterface, etc.)
- Delete operations (deleteApplication, deleteProcessModel, etc.)
- Add operations that reference existing objects (addRecordTypeField with existing relationship)

**Do NOT use for:**
- List operations (no UUID needed)
- Create operations (no UUID exists yet)
- Operations using name-based identifiers

### Why This Matters

**Problem:** User may provide stale UUIDs from previous sessions, documentation, or memory.

**Risk:** Operating on wrong object, causing data corruption or unintended changes.

**Solution:** Always verify UUID resolves to expected object before proceeding.

### Workflow Steps

1. **Receive UUID from user or prior operation**
   - Explicit: User types UUID in request
   - Implicit: UUID from earlier in conversation
   - Tool output: UUID returned from create/get operation earlier

2. **Call get operation to verify UUID**
   - Call appropriate `get` tool with UUID
   - Examples: `getRecordType(uuid)`, `getApplication(uuid)`, `getInterface(uuid)`
   - Handle errors: UUID not found, permission denied

3. **Extract human-readable identifier**
   - Name, type, key metadata
   - Creation date, last modified (if available)
   - Application context (if relevant)

4. **Present to user for confirmation**
   ```
   Found [ObjectType] "[Name]" (UUID: [uuid])
   Created: [date]
   Application: [appName]
   
   Is this the correct [ObjectType]? (yes/no)
   ```

5. **Proceed or abort based on response**
   - Yes → Continue with operation
   - No → Ask user to provide correct UUID or name

6. **Document in conversation**
   - Store UUID-to-name mapping for session
   - Reduces future verification prompts for same object

### Error Handling

**UUID not found:**
```
UUID [uuid] not found in the environment.

This may mean:
- The object was deleted
- The UUID is from a different environment
- The UUID is incorrect

Please provide the object name or verify the UUID.
```

**Multiple objects match (shouldn't happen, but defensive):**
```
UUID [uuid] matched multiple objects (this is unexpected).

Found:
- [ObjectType1] "[Name1]"
- [ObjectType2] "[Name2]"

Please verify which object you want to modify.
```

### When to Skip Verification

**Safe to skip if:**
- UUID was just created in this conversation (within last 5 exchanges)
- UUID was verified within last 10 exchanges
- User explicitly says "use the UUID from X" where X is recent and unambiguous

**Example of safe skip:**
```
User: "Create a Case record type"
AI: [calls createRecordType, gets UUID abc-123]
User: "Add a status field to that record type"
AI: [uses UUID abc-123 without re-verifying, since it was just created]
```

### Cross-References

For object-specific UUID usage patterns, see:
- UUID format and generation: `references/tools-mcp.md`
- Record Type UUIDs: `references/record-types.md`
- Application UUIDs: `references/applications.md`

---

<a name="universal-workflow-4-ambiguous-request-clarification"></a>
## Universal Workflow 4: Ambiguous Request Clarification

### When to Use

Apply this workflow when user request is missing required information:
- Create request without name
- Update request without specifying which object
- Operation with multiple possible interpretations
- Reference to "that" or "it" when context is unclear

### Common Ambiguity Patterns

| Pattern | Example Request | Missing Information |
|---|---|---|
| **No name** | "Create a record type" | What should it be named? |
| **Unclear target** | "Delete that field" | Which field? (multiple candidates) |
| **Ambiguous scope** | "Update the status" | Update field? Update value? Update record type? |
| **Incomplete parameters** | "Add a relationship to User" | What relationship name? MANY_TO_ONE or ONE_TO_MANY? |
| **Multiple candidates** | "Show me the interface" | Which interface? (app has 5 interfaces) |

### Workflow Steps

1. **Detect missing required parameter**
   - Parse user request
   - Check against tool schema required parameters
   - Identify what's missing

2. **Ask specific question**
   - Focus on ONE missing parameter at a time
   - Provide context for why you're asking
   - Suggest options if possible
   
   **Good:**
   ```
   What would you like to name the new record type?
   
   Suggestion: Use a noun that describes the entity (e.g., "Case", "Customer", "Asset")
   ```
   
   **Bad:**
   ```
   I need more information. What do you want to do?
   [Too generic — user already told you what they want]
   ```

3. **Provide helpful context**
   - Explain constraints (naming conventions, character limits)
   - Show examples from current application
   - Reference skill patterns if relevant

4. **Validate response**
   - Check against constraints (naming conventions, reserved words)
   - Verify uniqueness (if name collision detection applies)
   - Confirm format is correct

5. **Proceed with complete information**
   - If all parameters now available, execute operation
   - If more information needed, ask next question

### Examples by Scenario

**Scenario 1: No name provided**
```
User: "Create a record type for tracking cases"

AI: "What would you like to name the record type?

Per Appian conventions:
- Use singular noun (Case, not Cases)
- No spaces or special characters
- Start with capital letter

Suggestion based on your description: 'Case'"
```

**Scenario 2: Unclear target**
```
User: "Delete that status field"

[Context: Record type has both "status" and "caseStatus" fields]

AI: "I found two fields with 'status' in the name:
1. 'status' (TEXT field)
2. 'caseStatus' (TEXT field)

Which field do you want to delete? (1 or 2)"
```

**Scenario 3: Incomplete relationship**
```
User: "Add a relationship to the User record type"

AI: "What would you like to name this relationship?

The name should describe the relationship from Case → User.

Examples:
- 'assignedTo' (if tracking assigned user)
- 'createdBy' (if tracking creator)
- 'reviewer' (if tracking reviewer)

What name fits your use case?"
```

**Scenario 4: Multiple candidates**
```
User: "Show me the interface"

[Context: Application has 5 interfaces]

AI: "This application has 5 interfaces:
1. CM_CaseForm (case entry form)
2. CM_CaseList (case grid)
3. CM_Dashboard (summary dashboard)
4. CM_SearchInterface (search page)
5. CM_ReportInterface (reporting page)

Which interface would you like to see? (1-5 or name)"
```

### When NOT to Ask

**Don't ask if you can infer safely:**
- Naming convention is obvious from context ("Create Case record type" → name is "Case")
- Only one candidate exists ("Delete the field" + only one field exists → delete it)
- User just created object and says "add X to it" → use UUID from create operation

**Example of safe inference:**
```
User: "Create a Case record type"
AI: [infers name is "Case", proceeds without asking]

User: "Create a record type for tracking project milestones"
AI: [not obvious if name should be "Milestone", "ProjectMilestone", or "Project Milestone" → asks]
```

**Scenario 5: Multiple record types, ambiguous sample data target**
```
User: [Creates Customer record type]
User: [Creates Address record type]
User: "Add sample data"

[Context: Two record types created, unclear which needs sample data]

AI: "You just created 2 record types: Customer and Address.

     Which would you like to add sample data to?
     1. Customer (entity table)
     2. Address (entity table)
     3. Both (Customer first, then Address)
     
     What would you like to do? (1/2/3)"
```

**Important:** Even if only one record type was created recently, if user says "add sample data" without naming the table AND there's a FK relationship with empty target, proceed directly to checking FK blocking (don't ask which table). Only ask when genuinely ambiguous (multiple candidates).

### Cross-References

For object-specific naming conventions and parameter requirements, see:
- Record Types: `references/record-types.md`
- Expression Rules: `references/expression-rules.md` (prefix patterns)
- Applications: `references/applications.md` (prefix patterns)

---

<a name="universal-workflow-5-proactive-completion-patterns"></a>
## Universal Workflow 5: Proactive Completion Patterns

### When to Use

Apply this workflow when the skill documentation says "MUST", "MANDATORY", "REQUIRED", or "ALWAYS":
- USER field created → Add MANY_TO_ONE relationship to SYSTEM_RECORD_TYPE_USER
- Foreign key field created → Add both MANY_TO_ONE (from FK table) and ONE_TO_MANY (to referenced table)
- Application created → Discover defaultObjects (folders, groups) before adding objects
- Record type created → Platform auto-generates UUID and default fields

**Critical Rule:** Do NOT ask user for confirmation on these steps. Complete them automatically.

### How to Identify Mandatory Patterns

**Look for these signals in skill references:**

| Signal | Example | Pattern Type |
|---|---|---|
| **"MUST"** | "USER fields MUST have relationship to SYSTEM_RECORD_TYPE_USER" | Mandatory relationship |
| **"MANDATORY"** | "Bidirectional relationships are MANDATORY for FK fields" | Required pattern |
| **"ALWAYS"** | "Always add ONE_TO_MANY from referenced table" | Automatic completion |
| **"REQUIRED"** | "Primary key field is REQUIRED" | Platform requirement |
| **"Platform auto-generates"** | "Platform auto-generates folders and groups" | Platform behavior |

**Look for workflow sequences in references:**
```
Step 1: Create USER field
Step 2: Add MANY_TO_ONE relationship to SYSTEM_RECORD_TYPE_USER
Step 3: Set commonProperties.relatedRecordName
```
→ All 3 steps are mandatory, complete automatically

### Workflow Steps

1. **Detect mandatory follow-up step**
   - Parse skill reference for "MUST"/"MANDATORY" requirements
   - Check if current operation triggers follow-up
   - Example: Creating USER field → triggers SYSTEM_RECORD_TYPE_USER relationship

2. **Complete step automatically**
   - Call required tool operation
   - Use skill guidance for parameters (relationship name, type, properties)
   - Do NOT ask user for confirmation

3. **Inform user what was done**
   - State what was completed and why
   - Provide context from skill reference
   - Example: "Added MANY_TO_ONE relationship to SYSTEM_RECORD_TYPE_USER (required for USER fields)"

### Common Mandatory Patterns

<a name="proactive-completion-user-field-relationships"></a>
#### Pattern 1: USER Field Relationships

**Trigger:** Creating a field with `type: "USER"` or `type: "MULTIPLE_USER"`

**Mandatory steps:**
1. Add MANY_TO_ONE relationship from this record type to SYSTEM_RECORD_TYPE_USER
2. Set relationship name to match field name (or descriptive variant)
3. Set `commonProperties.relatedRecordName` to field name

**Example:**
```
User: "Add an assignedTo field (USER type) to the Case record type"

AI actions (automatic, no asking):
1. Call addRecordTypeField(name: "assignedTo", type: "USER")
2. Call addRecordTypeRelationship(
     relationshipType: "MANY_TO_ONE",
     relatedRecordTypeUuid: SYSTEM_RECORD_TYPE_USER,
     name: "assignedTo",
     commonProperties: {relatedRecordName: "assignedTo"}
   )

AI response: "Added 'assignedTo' USER field to Case record type. Also added MANY_TO_ONE relationship to SYSTEM_RECORD_TYPE_USER (required for USER fields to display correctly)."
```

<a name="proactive-completion-bidirectional-relationships"></a>
#### Pattern 2: Bidirectional Relationships

**Trigger:** Creating a foreign key field or adding a MANY_TO_ONE relationship

**Mandatory steps:**
1. Add MANY_TO_ONE relationship from FK table to referenced table
2. Add ONE_TO_MANY relationship from referenced table to FK table
3. Ensure both relationships reference each other

**Example:**
```
User: "Add a customerId field to Case, referencing the Customer record type"

AI actions (automatic, no asking):
1. Call addRecordTypeField(name: "customerId", type: "INTEGER")
2. Call addRecordTypeRelationship(
     relationshipType: "MANY_TO_ONE",
     relatedRecordTypeUuid: customerUuid,
     name: "customer"
   )
3. Call addRecordTypeRelationship(
     recordTypeUuid: customerUuid,
     relationshipType: "ONE_TO_MANY",
     relatedRecordTypeUuid: caseUuid,
     name: "cases"
   )

AI response: "Added 'customerId' field and bidirectional relationships:
- Case → Customer (MANY_TO_ONE) via 'customer'
- Customer → Case (ONE_TO_MANY) via 'cases'

Both relationships are required for proper navigation and queries."
```

#### Pattern 3: Application Default Objects

**Trigger:** Creating an application

**Mandatory steps:**
1. Call getApplication to retrieve defaultObjects (folders, groups)
2. Store UUIDs for later use (when creating objects in application)
3. Inform user what was discovered

**Example:**
```
User: "Create a Project Management application"

AI actions (automatic, no asking):
1. Call createApplication(name: "Project Management", prefix: "PM")
2. Call getApplication(uuid: pmAppUuid) to get defaultObjects
3. Extract folder UUIDs (processModelFolder, ruleFolder, documentFolder)
4. Extract group names (PM Users, PM Administrators)

AI response: "Created 'Project Management' application (prefix: PM).

Auto-generated resources:
- Groups: PM Users, PM Administrators
- Folders: PM Process Models, PM Rules & Constants, PM Knowledge Center

Ready to add record types, interfaces, or other objects to this application."
```

#### Pattern 4: Primary Key Field

**Trigger:** Creating a record type with database source

**Mandatory steps:**
1. Include `id` field (INTEGER, PRIMARY_KEY) in fields array
2. Use exact name "id" (platform requirement)
3. Do NOT ask user if they want a primary key (always required)

**Example:**
```
User: "Create a Case record type with fields: title, description, status"

AI actions (automatic, no asking):
1. Call createRecordType(
     name: "Case",
     fields: [
       {name: "id", type: "INTEGER", primaryKey: true},
       {name: "title", type: "TEXT"},
       {name: "description", type: "TEXT"},
       {name: "status", type: "TEXT"}
     ]
   )

AI response: "Created 'Case' record type with fields:
- id (INTEGER, primary key) [auto-added, required by platform]
- title (TEXT)
- description (TEXT)
- status (TEXT)"
```

### When to Ask (Even Though It's Mandatory)

**Exception: Missing naming information**

If a mandatory step requires a name that you cannot infer, ask for the name ONLY:
- Relationship name (if not obvious from field name)
- Record action name (if user says "add an action" without specifying)

**Example:**
```
User: "Add a relationship from Case to a custom Reviewer record type"

AI: "What would you like to name the relationship from Case → Reviewer?

This will be used in expressions like: rv!record[recordType!Case.relationships.YOURNAME]

Suggestion: 'reviewer' or 'assignedReviewer'"

[User provides name]

AI actions (after getting name):
1. Add MANY_TO_ONE (Case → Reviewer)
2. Add ONE_TO_MANY (Reviewer → Case) [automatic, no asking for reverse name]
```

### Informing the User

**Pattern for automatic completion:**
```
[Action completed]. [What was done automatically]. [Why it was automatic].

Example:
"Added 'assignedTo' USER field. Also added MANY_TO_ONE relationship to SYSTEM_RECORD_TYPE_USER (required for USER fields to display correctly in interfaces)."
```

**Pattern for multiple automatic steps:**
```
[Action completed]. [List of automatic steps]:
- [Step 1] [reason]
- [Step 2] [reason]

Example:
"Added 'customerId' field. Also added bidirectional relationships:
- Case → Customer (MANY_TO_ONE) via 'customer' [required for FK field]
- Customer → Case (ONE_TO_MANY) via 'cases' [required for bidirectional navigation]"
```

### Pitfalls to Avoid

❌ **Don't ask for confirmation on mandatory steps:**
```
Bad: "I need to add a relationship to SYSTEM_RECORD_TYPE_USER. Should I proceed?"
[This is mandatory — don't ask]

Good: "Added MANY_TO_ONE relationship to SYSTEM_RECORD_TYPE_USER (required for USER fields)."
```

❌ **Don't treat mandatory patterns as optional:**
```
Bad: "Would you like me to add the reverse ONE_TO_MANY relationship?"
[Bidirectional relationships are mandatory — don't ask]

Good: "Added bidirectional relationships (both MANY_TO_ONE and ONE_TO_MANY are required)."
```

❌ **Don't skip mandatory steps to avoid tool calls:**
```
Bad: [Creates USER field, skips SYSTEM_RECORD_TYPE_USER relationship to save a tool call]
[USER field won't display correctly — this breaks the application]

Good: [Creates USER field, adds relationship automatically]
```

### Cross-References

For object-specific mandatory patterns, see:
- USER field relationships: `references/record-types.md` (USER Field Special Handling section)
- Bidirectional relationships: `references/record-types.md` (Relationship Patterns section)
- Application default objects: `references/applications.md` (Creation Workflow section)
- Primary key requirements: `references/data-modeling.md` (Schema Design section)

---

### Sample Data Offering (Informative Pattern)

**Pattern Type:** Neither confirmation nor proactive — this is INFORMATIVE

**When to use:** After successfully creating a database-backed record type

**Workflow:**

1. Record type creation completes successfully
2. Inform user (don't ask):
   ```
   ✅ Created [RecordTypeName] record type with [N] fields.
   
   Note: No sample data added. To add test records, you can say:
   - "Add sample data" (uses intelligent defaults: 15-20 records with realistic distribution)
   - "Add 15 test records" (specify exact quantity)
   - "Add realistic data" (applies full distribution patterns across categories/time/users)
   ```

3. If user responds with request for sample data:
   - **Check for ambiguity:** If user created multiple record types recently and says "add sample data" without specifying which, follow Universal Workflow 4 (Ambiguous Request Clarification) to ask which table
   - Follow decision rules in `record-types.md` (Sample Data Decision Rules section)
   - Generate realistic sample data based on field types
   - Insert records via `insertRecordData`
   - Inform: "✅ Added [N] sample records"

**Why informative (not interactive confirmation):**
- Doesn't block workflow with a question
- User may not need sample data immediately
- Provides awareness without creating false choice
- User can request if/when needed

**Why not proactive completion:**
- Sample data is NOT mandatory
- User might want specific data, not generic samples
- Empty record type is valid state

**Example:**

After creating Employee record type:
```
✅ Created JTA Employee record type with 8 fields (id, name, email, departmentId, hireDate, salary, createdBy, modifiedBy).

Note: No sample data added. To add test records, you can say:
- "Add sample data" (uses intelligent defaults: 15-20 records with realistic distribution)
- "Add 15 test records" (specify exact quantity)
- "Add realistic data" (applies full distribution patterns across categories/time/users)
```

User can respond:
- "Add 10 sample employees" → AI generates and inserts (uses count specified)
- "Add sample data" → AI uses intelligent defaults (15-20 records, applies distribution patterns)
- "Add realistic data" → AI applies full distribution patterns (variety across all dimensions)
- (No response) → Move on, no sample data
- "I'll add my own data" → Acknowledged, no action

**Cross-reference:** For decision rules on when to ask vs when to use defaults, see `record-types.md` (Sample Data Decision Rules section).

---

<a name="universal-workflow-6-rename-update-confirmation"></a>
## Universal Workflow 6: Rename/Update Confirmation

### When to Use

Apply this workflow before renaming or significantly updating any Appian object:
- Renaming groups, record types, fields, relationships
- Renaming expression rules, interfaces, constants
- Changing key identifiers that may be hard-coded in SAIL expressions
- Updating objects that may have name-based dependencies

### UUID vs Name-Based Objects

**Critical Distinction:** Most Appian objects are UUID-based internally, making renames SAFE. Only a few objects have name-based dependencies.

#### UUID-Based Objects (Renames are SAFE)

These objects use UUIDs internally. Renaming updates all system references automatically:

| Object Type | UUID Used For | Rename Impact | Manual Updates Needed |
|---|---|---|---|
| **Record Types** | Relationships, views, actions | ✅ SAFE - all references auto-update | None (unless hard-coded in expressions) |
| **Fields** | Relationships, title expressions | ✅ SAFE - UUID never changes | None (unless hard-coded in expressions) |
| **Folders** | Constants, document storage | ✅ SAFE - constants store UUID | None |
| **Documents** | Process models, interfaces | ✅ SAFE - references use UUID | None |
| **Process Models** | Constants, start forms | ✅ SAFE - constants store UUID | None |
| **Interfaces** | Sites, process models | ✅ SAFE - references use UUID | None |
| **Expression Rules** | Calls from other rules | ✅ SAFE - UUID-based calls | None (unless hard-coded in expressions) |

**Why these are safe:**
- Internal references use UUID, not name
- UUID never changes when you rename
- Appian automatically maintains referential integrity

#### Name-Based Objects (Renames Require Care)

Two object types have name-based dependencies:

| Object Type | Name Used For | Rename Impact | Risk Level | Manual Updates Needed |
|---|---|---|---|---|
| **Constants** | All `cons!NAME` references | ❌ HIGH - NOTHING auto-updates | ❌ HIGH | ALL expressions, interfaces, process models |
| **Groups** | SAIL expressions, API operations | ⚠️ MIXED - constants auto-update, expressions don't | ⚠️ MEDIUM | Hard-coded group names in SAIL code |

##### Constants (HIGH RISK)

**CRITICAL:** Constants ARE UUID-based internally, BUT expressions reference them by NAME (`cons!NAME`), not UUID.

**What BREAKS when renaming a constant:**
- ❌ ALL expression rules using `cons!OLD_NAME`
- ❌ ALL interfaces using `cons!OLD_NAME`
- ❌ ALL process models using `cons!OLD_NAME`

**What's PRESERVED:**
- ✅ Constant UUID (unchanged)
- ✅ Constant value (unchanged)
- ✅ Constant type (unchanged)

**Why HIGH RISK:**
Unlike groups (where GROUP constants auto-update), constant renames provide NO automatic updates. Every single reference must be manually updated.

**Detection:** Cannot programmatically detect all uses (would require parsing SAIL code in all objects).

**Recommendation:** Instead of renaming:
1. Create new constant with new name, same value
2. Gradually update expressions to use new constant
3. Delete old constant when all references updated

##### Groups (MEDIUM RISK)

**What auto-updates when renaming a group:**
- ✅ GROUP constants (value changes to new name)
- ✅ Group memberships (UUID-based, preserved)
- ✅ Parent-child hierarchy (UUID-based, preserved)

**What requires manual updates:**
- ❌ Hard-coded group names in `a!isUserMemberOfGroup()` calls
- ❌ Hard-coded group names in security expressions
- ❌ Hard-coded group names in process model conditions

### Workflow Steps

1. **Identify object type**
   - UUID-based → LOW RISK, continue to step 4 with LOW RISK confirmation
   - Constant → HIGH RISK, continue to step 2
   - Group → MEDIUM RISK, continue to step 3

2. **For Constants: Present CRITICAL risk warning**
   - Skip detection (cannot automatically detect all uses)
   - Proceed directly to step 4 with HIGH RISK confirmation

3. **For Groups: Search for hard-coded references**
   - Cannot detect automatically
   - Warn: "Search interfaces, expression rules, and process models for hard-coded group name strings"

4. **Present appropriate risk level:**

**For Constants (HIGH RISK):**
```
⚠️ CRITICAL: You are about to RENAME constant "[OLD_NAME]" to "[NEW_NAME]"

This will BREAK all expressions using cons!OLD_NAME

What breaks:
❌ ALL expression rules using cons!OLD_NAME
❌ ALL interfaces using cons!OLD_NAME
❌ ALL process models using cons!OLD_NAME

What's preserved:
✅ Constant UUID ([uuid])
✅ Constant value ("[value]")
✅ Constant type ([type])

Manual updates required:
1. Search ALL expression rules for "cons!OLD_NAME"
2. Search ALL interfaces for "cons!OLD_NAME"
3. Search ALL process models for "cons!OLD_NAME"
4. Update EACH reference to "cons!NEW_NAME"

⚠️ No automatic updates (unlike group rename where GROUP constants auto-update)

Alternative approach (RECOMMENDED):
1. Create new constant "NEW_NAME" with same value
2. Gradually migrate expressions to cons!NEW_NAME
3. Delete old constant when all references updated

This avoids breaking everything at once.

Type 'RENAME [OLD_NAME]' to confirm HIGH RISK operation.
```

**For UUID-based objects (LOW risk):**
```
Rename "[ObjectType]" from "[OldName]" to "[NewName]"?

✅ All system references will automatically update (UUID-based).
❌ Any hard-coded "[OldName]" strings in SAIL expressions will need manual updates.

Continue? (yes/no)
```

**For Groups (MEDIUM risk):**
```
Rename group "[OldName]" to "[NewName]"?

What Appian will automatically update:
✅ GROUP constants (value will change to "[NewName]")
✅ Group memberships (all members preserved)
✅ Parent-child relationships (hierarchy preserved)

What requires manual updates:
❌ Hard-coded "[OldName]" in SAIL security expressions
❌ Hard-coded "[OldName]" in expression rules
❌ Hard-coded "[OldName]" in process model conditions

Recommendation: After rename, search for "[OldName]" and update hard-coded references.

Continue? (yes/no)
```

5. **Wait for user confirmation**
   - HIGH RISK (constants): Wait for typed confirmation matching exact old name
   - MEDIUM/LOW RISK: Wait for yes/no response
   - If user declines, abort operation

6. **Execute rename** (only after confirmation)
   - Call `update` operation with new name
   - UUID remains unchanged

7. **Inform user of next steps:**

**For UUID-based objects:**
```
✅ Renamed "[ObjectType]" from "[OldName]" to "[NewName]"

All system references automatically updated.

Optional: Search for hard-coded "[OldName]" strings in SAIL expressions.
```

**For Groups:**
```
✅ Group renamed from "[OldName]" to "[NewName]"

Automatic updates:
- GROUP constants updated
- Memberships preserved
- Hierarchy preserved

Manual action required:
Search for hard-coded "[OldName]" in:
1. Interfaces (security expressions)
2. Expression rules (a!isUserMemberOfGroup calls)
3. Process models (group name conditions)
```

### Common Rename Scenarios

#### Scenario 1: Renaming a Record Type

**User:** "Rename the Case record type to Support Case"

**AI Workflow:**
1. Identify: UUID-based object → LOW RISK
2. Present: "Rename record type 'Case' to 'Support Case'? All relationships, views, and actions will automatically update. Continue? (yes/no)"
3. Wait for user confirmation (yes/no)
4. Execute: `updateRecordType(uuid, name: "Support Case")`
5. Inform: "✅ Renamed to 'Support Case'. All system references automatically updated."

No follow-up needed unless hard-coded references exist.

#### Scenario 2: Renaming a Group

**User:** "Rename CM Administrators to CM Admin Group"

**AI Workflow:**
1. Identify: Group → MEDIUM RISK
2. Search: Cannot auto-detect hard-coded references
3. Present: "Rename group 'CM Administrators' to 'CM Admin Group'? GROUP constants will auto-update, but hard-coded references in SAIL need manual updates. Continue? (yes/no)"
4. Wait for user confirmation (yes/no)
5. Execute: `updateGroup(groupName: "CM Administrators", name: "CM Admin Group")`
6. Inform: "✅ Renamed. GROUP constants updated. Search for 'CM Administrators' in SAIL expressions and update manually."

Follow-up: User searches and updates hard-coded references.

#### Scenario 3: Renaming a Field

**User:** "Rename the status field to caseStatus"

**AI Workflow:**
1. Identify: UUID-based object → LOW RISK
2. Check: Is field used in relationships, title expression, or views? (structural check)
3. Present: "Rename field 'status' to 'caseStatus'? Used in 1 relationship and title expression. All references will auto-update. Continue? (yes/no)"
4. Wait for user confirmation (yes/no)
5. Execute: `updateRecordTypeField(uuid, fieldName: "caseStatus")`
6. Inform: "✅ Renamed. Relationship and title expression automatically updated."

No follow-up needed.

### Pitfalls to Avoid

- **Assuming renames are risky** — Most objects are UUID-based and safe to rename
- **Not distinguishing groups from other objects** — Groups are the exception, not the rule
- **Over-warning** — Don't warn about "breaking references" for UUID-based objects
- **Under-warning for groups** — Do warn about hard-coded SAIL expressions

### Cross-References

For object-specific rename considerations:
- Groups: `references/security-patterns.md` (Group Rename section)
- Record Types: `references/record-types.md` (Name Collision Detection section)
- Expression Rules: `references/expression-rules.md` (Naming Conventions section)

---

<a name="universal-workflow-7-dependency-checking"></a>
## Universal Workflow 7: Dependency Checking

**⚠️ CRITICAL:** This workflow is REQUIRED for all DELETE and RENAME operations. 
- Universal Workflow 1 (Delete Confirmation) MUST use this workflow's templates
- Universal Workflow 6 (Rename Confirmation) MUST use this workflow's templates

### When to Use

Before any destructive or breaking operation:
- **Delete operations** — What depends on this object?
- **Rename operations** (name-based objects) — What references this name?
- **Input removal** (expression rules, interfaces) — What calls this with that parameter?
- **Breaking changes** — What will stop working?

**Key Principle:** Know what breaks BEFORE the user confirms the operation.

---

### Why Dependency Checking Matters

**Without dependency checking:**
- User deletes Status record type → all Issue records lose their status field reference
- User renames constant PMS_ADMIN_GROUP → all expressions using `cons!PMS_ADMIN_GROUP` break
- User removes input from expression rule → all calling rules fail

**With dependency checking:**
- Present impact BEFORE user confirms
- Offer alternatives (cascade delete, refactor, cancel)
- Document what needs manual updates

---

### Current Tool Limitations

⚠️ **Important:** The current MCP tools support only **structural dependency checks**. Expression-based dependency detection (e.g., "which expressions reference this constant?") is **not available**.

**What we CAN check (structural):**
- Record type relationships (MANY_TO_ONE, ONE_TO_MANY)
- Record type views and actions
- Record type user filters
- Record data existence
- Group hierarchy (parent/child relationships)
- Group members
- Application contained objects

**What we CANNOT check (requires tools not available):**
- Which expressions reference a constant
- Which interfaces hard-code a group name
- Which process models reference an expression rule
- Which expressions call another expression rule
- Hard-coded string references in SAIL code

**Approach:** Use structural checks + manual verification fallback.

---

### Workflow Steps

#### Step 1: Identify Operation Type

Determine what kind of operation is being attempted:

| Operation | Dependency Type | Check Method |
|---|---|---|
| Delete record type | Relationships, views, actions, data | Structural (available) |
| Delete field | Related views/actions using field | Structural (available) |
| Delete constant | Expression references | Manual fallback (not available) |
| Delete expression rule | Calling rules/interfaces | Manual fallback (not available) |
| Delete group | Security expressions, hierarchy | Structural + manual |
| Rename constant | `cons!NAME` references | Manual fallback (not available) |
| Rename group | Hard-coded name strings | Structural + manual |
| Remove rule input | Calling rules with that parameter | Manual fallback (not available) |

#### Step 2: Perform Available Structural Checks

Use available MCP tools to check structural dependencies:

**For Record Types:**
```
1. Check relationships: Does this record type have relationships to/from other types?
   Tool: getRecordType(uuid) → check relationships array
   
2. Check views: Does this record type have custom views?
   Tool: listRecordTypeViews(recordTypeUuid)
   
3. Check actions: Does this record type have actions?
   Tool: listRecordTypeActions(recordTypeUuid)
   
4. Check data: Does this record type have data?
   Tool: listRecordData(recordTypeUuid, limit=1)
```

**For Groups:**
```
1. Check hierarchy: Does this group have child groups?
   Tool: listGroups() → filter by parentGroupUuid
   
2. Check members: Does this group have members?
   Tool: listGroupMembers(groupUuid)
   
3. Check constants: Does a GROUP constant reference this group?
   Tool: listConstants() → filter by type=GROUP, check values
```

**For Applications:**
```
1. Check contained objects: What objects are in this application?
   Tool: listApplicationObjects(applicationUuid)
```

#### Step 3: Document Manual Verification Requirements

For expression-based dependencies (not detectable via tools), present manual verification guidance:

**Template for manual fallback:**
```
⚠️ Cannot automatically detect all dependencies.

Manual verification required:
1. Search expression rules for "[OBJECT_NAME]" or "cons!CONSTANT_NAME"
2. Search interfaces for hard-coded references
3. Search process models for script task expressions
4. Check Web APIs for usage in request/response expressions

Use Appian Designer's "Find Usages" feature:
- Open object in Designer
- Right-click → "Find Usages" (if available for object type)
- Review usage list before proceeding
```

#### Step 4: Present Dependencies to User

Show structural dependencies + manual verification requirements BEFORE confirmation:

**Template for presenting dependencies:**
```
⚠️ [OPERATION] will impact:

Structural dependencies (confirmed):
❌ 3 relationships from other record types
❌ 2 custom views
❌ 4 record actions
❌ 15,432 existing records

Manual verification required (cannot detect automatically):
⚠️ Expressions using cons!STATUS_OPEN
⚠️ Interfaces hard-coding "Open" status
⚠️ Process models checking status field

What will happen:
- [List specific impacts]

Options:
1. Cancel (recommended until manual verification complete)
2. Proceed with [OPERATION] (you are responsible for manual updates)
3. [Alternative approach if available]

What would you like to do? (1/2/3)
```

#### Step 5: Offer Resolution Strategies

Based on operation type and dependencies found:

**Safe cascade operations (offer if applicable):**
- Delete record type → offer cascade delete related data (if updateTable=true supported)
- Delete parent group → offer cascade delete child groups (if supported)

**Refactoring alternatives (recommend):**
- Instead of deleting constant → deprecate and create replacement
- Instead of renaming constant → create new, migrate references, delete old
- Instead of removing input → add new input, deprecate old (non-breaking change)

**Manual update guidance:**
- List specific files/objects requiring manual updates
- Suggest search patterns for finding references
- Document post-operation verification steps

---

### Dependency Check Examples

#### Example 1: Record Type Delete with Structural Dependencies

**Operation:** Delete "PMS Status" record type

**Structural checks performed:**
```
✓ getRecordType(uuid) → relationships: 2 MANY_TO_ONE from other types
✓ listRecordTypeViews(uuid) → views: 1 custom view
✓ listRecordTypeActions(uuid) → actions: 0
✓ listRecordData(uuid, limit=1) → data: 4 records exist
```

**Presentation to user:**
```
⚠️ Cannot delete PMS Status: dependencies found

Structural dependencies (confirmed):
❌ 2 record types reference Status:
   - PMS Project (relationship: status)
   - PMS Task (relationship: status)
❌ 1 custom view: Status Grid
❌ 4 status records exist (Created, Assigned, Approved, Closed)

Impact:
- Deleting Status will break Project and Task relationships
- 15 projects and 43 tasks currently reference these statuses
- Status Grid view will fail to load

Options:
1. Cancel (recommended - dependencies must be removed first)
2. Remove relationships from Project and Task, then retry
3. Keep Status record type

What would you like to do? (1/2/3)
```

#### Example 2: Constant Delete with Manual Fallback

**Operation:** Delete constant "PMS_ADMIN_GROUP"

**⚠️ IMPORTANT:** For constant deletions, use the [Constant Delete Special Case](#constant-delete-special-case) template which provides the complete structure. This example shows the workflow; the Special Case provides the template.

**Structural checks performed:**
```
✓ getConstant(uuid) → type: GROUP, value: "PMS Administrators"
✗ Cannot detect expressions using cons!PMS_ADMIN_GROUP (tool not available)
```

**Presentation to user:**
```
⚠️ CRITICAL: You are about to DELETE constant "PMS_ADMIN_GROUP"

Type: GROUP
Value: "PMS Administrators"
UUID: [uuid]

⚠️ Cannot automatically detect all dependencies.

Manual verification REQUIRED before proceeding:
1. Search ALL expression rules for "cons!PMS_ADMIN_GROUP"
2. Search ALL interfaces for "PMS_ADMIN_GROUP" 
3. Search ALL process models for security expressions
4. Search ALL record actions for security/visibility expressions
5. Check Web API security expressions

Use Appian Designer:
- Open Constants page
- Right-click "PMS_ADMIN_GROUP" → "Find Usages"
- Review ALL usage locations

What breaks if you proceed:
❌ ALL expressions using cons!PMS_ADMIN_GROUP will fail
❌ Security checks will break (may grant unauthorized access)
❌ Process assignments may fail

Options:
1. Cancel (recommended until manual verification complete)
2. Proceed (you are responsible for updating ALL references)

Type 'DELETE PMS_ADMIN_GROUP' to confirm HIGH RISK operation.
```

#### Example 3: Group Hierarchy Check

**Operation:** Delete group "PMS Developers"

**Structural checks performed:**
```
✓ listGroups() → child groups: 2 found
   - PMS Senior Developers
   - PMS Junior Developers
✓ listGroupMembers(uuid) → members: 8 users
✓ listConstants() → GROUP constants: 1 found (PMS_DEVELOPERS_GROUP)
```

**Presentation to user:**
```
⚠️ Cannot delete PMS Developers: dependencies found

Structural dependencies (confirmed):
❌ 2 child groups (will be orphaned):
   - PMS Senior Developers (3 members)
   - PMS Junior Developers (5 members)
❌ 8 direct members
❌ 1 GROUP constant: PMS_DEVELOPERS_GROUP

Manual verification required:
⚠️ Security expressions using a!isUserMemberOfGroup()
⚠️ Hard-coded "PMS Developers" in interfaces/rules
⚠️ Process model security expressions

Impact:
- Child groups will lose parent (hierarchy broken)
- GROUP constant value will become invalid
- Security expressions using the constant will fail

Options:
1. Cancel (recommended)
2. Remove child groups first, then retry
3. Reassign child groups to different parent

What would you like to do? (1/2/3)
```

---

### Cascade Delete Considerations

Some operations support cascade deletes. Present cascade options ONLY when safe and supported by tools.

**When cascade is SAFE:**
- Delete record type with `updateTable=true` → drops database table and data
- Delete parent folder → deletes contained documents (if supported)

**When cascade is UNSAFE:**
- Delete record type → DO NOT cascade delete related record types (breaks other relationships)
- Delete group → DO NOT cascade delete child groups (breaks member hierarchy)
- Delete constant → NO CASCADE (expressions must be manually updated)

**Template for offering cascade:**
```
Cascade delete option available:

⚠️ Delete record type AND drop database table?

This will:
✅ Remove record type metadata
✅ Drop PMS_STATUS table (4 records deleted)
❌ Break 2 relationships (Project.status, Task.status)

Cascade delete is IRREVERSIBLE.

Options:
1. Delete record type only (preserve table and data)
2. Delete record type AND table (IRREVERSIBLE)
3. Cancel

What would you like to do? (1/2/3)
```

---

### Manual Verification Guidance Templates

#### For Constants (Expression References)

```
⚠️ Manual verification required for constant: [CONSTANT_NAME]

Search for ALL references in Appian Designer:
1. Expression Rules:
   - Open "Expression Rules" page
   - Use search: "cons![CONSTANT_NAME]"
   - Review all matching rules
   
2. Interfaces:
   - Open "Interfaces" page
   - Use search: "[CONSTANT_NAME]"
   - Check expression editor for each match
   
3. Process Models:
   - Open "Process Models" page
   - Check Script Task expressions
   - Check XOR Gateway conditions
   - Check start form expressions
   
4. Record Types:
   - Check record action visibility expressions
   - Check record view expressions
   - Check user filter expressions

Expected locations:
- Security expressions: a!isUserMemberOfGroup(user, cons![CONSTANT_NAME])
- Status checks: rv!record.status = cons![CONSTANT_NAME]
- Conditional logic: if(pv!status = cons![CONSTANT_NAME], ...)

Document all findings before proceeding.
```

#### For Expression Rules (Calling Dependencies)

```
⚠️ Manual verification required for expression rule: [RULE_NAME]

Find all calling locations:
1. Right-click rule in Designer → "Find Usages"
2. Review list of:
   - Other expression rules calling this rule
   - Interfaces using this rule
   - Process models calling this rule
   
If "Find Usages" not available:
1. Search expression rules for "rule![RULE_NAME]"
2. Search interfaces for "[RULE_NAME]"
3. Check process model Script Tasks

Document impact:
- How many rules call this?
- Which interfaces use this?
- Which process models depend on this?

If removing input parameter:
- All calling locations must be updated
- Calls with removed parameter will fail
```

---

### When to Skip Dependency Checking

**Skip structural checks when:**
- User explicitly requests quick delete (acknowledged risks)
- Object was just created in this session (no dependencies possible)
- Object type has no dependency mechanisms (e.g., folders with no contents)

**Never skip manual verification guidance for:**
- Constants (expression references)
- Expression rules (calling dependencies)
- Name-based references (groups hard-coded in expressions)

---

### Cross-References

**Related Universal Workflows:**
- Universal Workflow 1: Delete Confirmation (uses dependency checking results)
- Universal Workflow 6: Rename Confirmation (uses dependency checking for name-based objects)

**Object-Specific Dependency Checks:**
- Record Types: `references/record-types.md` (Deletion Confirmation section)
- Expression Rules: `references/expression-rules.md` (Dependency Management section)
- Applications: `references/applications.md` (Application Deletion section)
- Groups: `references/security-patterns.md` (Group Management section)

**Tool Limitations:**
- Phase 1.5 documents tool enhancement proposals for expression-based dependency detection
- See `/my-docs/interactive-guidance/tool-change-proposal-dependency-detection.md` (not in skill)

---

## When You Need More

This file covers **universal patterns** that apply across all object types. For **object-specific confirmation patterns**, load the relevant domain reference:

| Object Type | Reference File | Object-Specific Patterns |
|---|---|---|
| Record Types | `references/record-types.md` | Field dependency checks, relationship validation, view/action impacts |
| Expression Rules | `references/expression-rules.md` | Rule dependency graph (what calls this rule) |
| Applications | `references/applications.md` | Contained object enumeration, cascade delete impacts |
| Process Models | `references/process-models.md` | Node dependency checks, active instance warnings |
| Groups | `references/supporting-objects.md` | Member enumeration, child group impacts, security role checks |

**Loading guidance:**
1. Always load this file (confirmation-patterns.md) first
2. Then load object-specific reference for additional confirmation logic
3. Combine universal + object-specific patterns in your workflow
