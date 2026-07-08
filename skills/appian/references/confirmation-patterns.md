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

**Other Guidance**
- [Note on Renames](#note-on-renames)

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

Apply this workflow before ANY DELETE operation:
- Deleting applications, record types, fields, relationships, views, actions
- Deleting expression rules, interfaces, constants, process models
- Deleting groups, folders, documents, sites, connected systems, Web APIs
- Removing rule inputs or making breaking changes

**Key Principle:** Know what breaks BEFORE the user confirms the operation.

---

### Why Dependency Checking Matters

**Without dependency checking:**
- User deletes Status record type → all Issue records lose their status field reference
- User deletes constant → all expressions using `cons!CONSTANT_NAME` break
- User removes input from expression rule → all calling rules fail

**With dependency checking:**
- Present impact BEFORE user confirms
- Offer alternatives (cascade delete, refactor, cancel)
- Document what needs manual updates

---

### Tool Capabilities (Updated 2026-07-02)

✅ **Expression-based dependency detection now available** via `getObjectDependents` MCP tool.

**What we CAN check:**

1. **Structural dependencies:**
   - Record type relationships (MANY_TO_ONE, ONE_TO_MANY)
   - Record type views, actions, user filters
   - Record data existence
   - Group hierarchy and members
   - Application contained objects

2. **Expression-based dependencies:**
   - Which expressions reference a constant (cons!)
   - Which rules/interfaces/Web APIs call an expression rule (rule!)
   - Which objects reference a record type (recordType!)
   - Which process models use record types in nodes/variables
   - Which sites use interfaces as pages
   - Exact line numbers where references occur

**How it works:**
- `getObjectDependents(uuid)` returns all design objects referencing the target
- Response includes: dependent UUID, type, name, and breadcrumb showing exact location
- Supports: Constants, Expression Rules, Record Types, Interfaces, Process Models, Web APIs, Sites, Connected Systems
- Same object can appear multiple times (one entry per reference location)
- Performance: < 2 seconds for 300+ dependents

**What we still CANNOT check:**
- Field-level dependencies (fields referenced by name, not UUID)
- Hard-coded string references (group names, status values)
- Document/folder references in expressions
- Integration configurations

---

### Risk Levels

| Risk Level | Scope | Examples | Confirmation Type |
|---|---|---|---|
| **CRITICAL** | Affects multiple objects or production data | Application delete, Record Type delete, Process Model delete, **Constant delete**, **Expression Rule breaking changes** (input removal, type changes) | Typed confirmation or 3-option choice |
| **HIGH** | Affects single object with potential dependencies | Field delete, Relationship delete, Expression Rule delete (no callers) | Yes/No confirmation |
| **MEDIUM** | Affects single object, limited dependencies | View delete, Action delete | Yes/No confirmation |

**Notes:** 
- Constants are CRITICAL because they're name-based (`cons!NAME`) not UUID-based. Deleting breaks ALL expressions using that constant with no automatic updates.
- Expression Rule breaking changes (input removal, input type change, return type change) are CRITICAL because they break ALL callers with no automatic updates.

---

### Workflow Steps

⚠️ **IMPORTANT:** Steps 4-6 describe your internal process. Only Step 7's output is shown to the user.

**What user sees:** Final dependency presentation (Step 7 template)  
**What user does NOT see:** "Step 4:", "✓ getObjectDependents()", "Calling...", processing details

---

#### Step 1: Receive Delete Request

- User may provide name, UUID, or description
- May be explicit ("delete X") or implicit ("remove that")

---

#### Step 2: Verify Object Existence

- Call appropriate `get` or `list` operation to resolve identifier
- If UUID provided, verify it's valid
- If name provided, resolve to UUID
- If ambiguous, ask for clarification

---

#### Step 3: Extract Object Details

- Name, type, creation date, last modified
- Key metadata (number of fields, relationships, dependencies)
- Application context if applicable

---

#### Step 4: Identify Operation Type

Determine what kind of operation is being attempted:

| Operation | Dependency Type | Check Method |
|---|---|---|
| **Phase 1A (Validated):** | | |
| Delete constant | Expression references | Expression (Step 5) ✅ |
| Delete expression rule | Calling rules/interfaces | Expression (Step 5) ✅ |
| Delete record type | Relationships, views, actions, data, expressions | Expression (Step 5) + Structural (Step 6) ✅ |
| **Phase 1B (Testing Required):** | | |
| Delete interface | Calling objects, sites using it | Expression (Step 5) - validate in testing |
| Delete process model | Calling objects | Expression (Step 5) - validate in testing |
| Delete Web API | Calling expressions | Expression (Step 5) - validate in testing |
| Delete connected system | Integrations using it | Expression (Step 5) ✅ |
| **Structural Only:** | | |
| Delete field | Related views/actions using field | Structural (Step 6 + see record-types.md) |
| Delete group | Hierarchy, members | Structural (Step 6) + manual |
| Delete application | Contained objects | Structural (Step 6) |
| **Manual Fallback:** | | |
| Remove rule input | Calling rules with that parameter | Manual fallback - parameter-level not supported |

---

#### Step 5: Check Expression Dependencies

**CRITICAL: ALWAYS call `getObjectDependents` first. Do NOT skip to manual fallback unless the tool call fails.**

**Process:**

1. **Call `getObjectDependents(uuid)` - MANDATORY FIRST STEP**
   - Returns list of dependents with uuid, type, name, breadcrumb
   - Same object may appear multiple times (one per reference location)
   - Supported for: constants, expression rules, interfaces, process models, Web APIs, connected systems, record types, applications, sites

2. Deduplicate and group:
   - Group by `type` (APPLICATION, CONSTANT, FREEFORM_RULE, INTERFACE, OUTBOUND_INTEGRATION, PROCESS_MODEL, RECORD_TYPE, SITE, WEB_API)
   - Within each type, deduplicate by `uuid` to count unique objects
   - Keep first 5 breadcrumbs per object for presentation

3. Present:
   - If < 10 unique objects per type: show all
   - If 10+ unique objects per type: show first 10 returned + "...and N more (type 'details' for full list)"
   - Breadcrumb format: "Interface Definition: Lines 19, 180, 186, 108, 109..." (first 5, then "...")
   - Note: Tool returns objects in arbitrary order (not sorted by reference count)

**ONLY if getObjectDependents call fails or returns an error:**

Use manual verification fallback below. Do NOT use this fallback if the tool call succeeded (even if it returned zero dependencies).

**Manual Verification Fallback (ONLY if getObjectDependents fails):**

```
⚠️ Automatic dependency check failed.

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

---

#### Step 6: Perform Structural Checks (When Applicable)

**For Record Types:**
- Relationships: getRecordType(uuid).relationships
- Views: listRecordTypeViews(uuid)
- Actions: listRecordTypeActions(uuid)
- Data: listRecordData(uuid, limit=1)

**For Groups:**
- Hierarchy: listGroups() filtered by parentGroupUuid
- Members: listGroupMembers(uuid)
- Constants: listConstants() filtered by type=GROUP, check values

**For Applications:**
- Contained objects: listApplicationObjects(uuid)

**For Fields (record type fields):**
- Relationships: Check if field is sourceRecordTypeFieldUuid or targetRecordTypeFieldUuid in getRecordType(recordTypeUuid).relationships
- Title expression: Check if titleExpression in getRecordType(recordTypeUuid) references field UUID
- Views: Check if field displayed in listRecordTypeViews(recordTypeUuid)
- Note: Cannot check expression-based dependencies (fields referenced by name: recordType!RT.fields.fieldName, not by UUID)

---

#### Step 7: Present Dependencies to User

Present the final result using templates below. This is what the USER sees.

**Do NOT show to user:**
- Step numbers ("Step 5:", "Step 6:")
- Tool calls ("✓ getObjectDependents(uuid) →", "Calling getRecordType...")
- Processing details ("Deduplicating...", "Checking...")

**DO show to user:**
- Clean dependency list grouped by type
- Clear impact statement ("will break N objects")
- Action options (Cancel/Proceed/Details)

**Use object-specific templates when available:**

For these operations, use the specialized templates (more context, object-specific warnings):
- **Constants** → Use [Constant Delete Special Case](#constant-delete-special-case) template
- **Expression Rules (delete)** → Use [Expression Rule Delete Special Case](#expression-rule-delete-special-case) template
- **Expression Rules (breaking changes)** → Use [Expression Rule Breaking Changes Special Case](#expression-rule-breaking-changes-special-case) template
- **Record Types** → Use [Record Type Delete Special Case](#record-type-delete-special-case) template

For all other operations, use generic template below.

**Generic user-facing template:**
```
⚠️ [OPERATION] will impact N design objects:

Design object dependencies:
❌ [Type]: N unique (M references)
   - [Breadcrumb examples - see variations below]
[Repeat for each type found]

[If record types/groups: add Structural dependencies section]

Impact: [What breaks - be specific]

Options:
1. Cancel (recommended - update N dependencies first)
2. Proceed (you fix N broken references)
3. [If 10+ dependents: add "Details (show full list)"]
4. [Alternative if available]

What would you like to do? (1/2/3/4)
```

**Variations:**

**< 10 unique objects per type:**
```
❌ Interfaces: 2
   - Interface Definition: Line 19
   - Interface Definition: Line 108
```

**10+ unique objects per type:**
```
❌ Interfaces: 13 unique (322 references)
   Top references:
   - Interface Definition: Lines 19, 180, 186, 108, 109...
   - Interface Definition: Lines 115, 116, 117, 118, 119...
   [Show up to 10 unique objects]
   ...and 3 more (type 'details' for full list)
```

**Record types - add after expression dependencies:**
```
Structural dependencies:
❌ 2 relationships from other record types
❌ 1 custom view
❌ 4 record actions
❌ 15,432 existing records
```

**If getObjectDependents fails:**
```
⚠️ Automatic dependency check unavailable: getObjectDependents tool failed.

Falling back to manual verification:
1. Search expression rules for "[OBJECT_NAME]"
2. Use Appian Designer "Find Usages"

[If applicable: Structural checks: ...]

Continue with manual verification? (yes/no)
```

**If user requests "details":**

When user types "details" after seeing "...and N more" in presentation:

1. Show all unique objects (remove top 10 limit)
2. Show all breadcrumbs per object (remove first 5 limit)
3. Use same template structure, just not truncated
4. Example:
   ```
   Full dependency list:
   
   ❌ Interfaces: 13 unique (322 references)
      - Interface 1: Lines 19, 180, 186, 108, 109, 115, 116, 117, 118, 119, 120, 121...
      - Interface 2: Lines 47, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64...
      [All 13 interfaces shown with all breadcrumbs]
   
   ❌ Expression Rules: 1 unique (23 references)
      - Expression Rule: Lines 61, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23...
   ```

---

#### Step 8: Offer Resolution Strategies

Based on operation type and dependencies found:

**Safe cascade operations (offer if applicable):**
- Delete record type → offer cascade delete related data (if updateTable=true supported)
- Delete parent group → offer cascade delete child groups (if supported)

**Refactoring alternatives (recommend):**
- Instead of deleting constant → deprecate and create replacement
- Instead of removing input → add new input, deprecate old (non-breaking change)

**Manual update guidance:**
- List specific files/objects requiring manual updates
- Suggest search patterns for finding references
- Document post-operation verification steps

---

#### Step 9: Get User Confirmation

**MANDATORY: ALWAYS ask for user confirmation. DO NOT skip this step.**

Present all findings from Step 7 (dependency check results) and Step 8 (resolution strategies if applicable), then **STOP and WAIT for user response**.

**Confirmation format by object type:**

**CRITICAL risk** (Application, Record Type, Process Model, Constant, Connected System):
- Require typed confirmation: `Type 'DELETE [Name]' to confirm this operation.`
- Must match exact name
- **Required even if zero dependencies found**

**HIGH/MEDIUM risk** (Expression Rule, Interface, Web API, Site):
- Simple confirmation: `Proceed with deletion? (yes/no)`
- **Required even if zero dependencies found**

**IMPORTANT:** The risk level is determined by OBJECT TYPE, not dependency count. Even objects with zero blocking dependencies require confirmation because:
- User may have changed their mind after seeing the analysis
- Deletion is permanent and irreversible
- This provides a final checkpoint before destructive action

**If user declines:**
- Abort operation
- Do not proceed to Step 10
- Inform user: "Deletion cancelled."

---

#### Step 10: Execute Delete Operation

Only after user confirms:
- Call appropriate MCP delete tool
- Inform user of success or failure

**Success message:**
```
✅ Deleted [ObjectType] "[Name]"

[If dependencies existed: Remind user to update N dependent objects]
```

**Failure message:**
```
❌ Failed to delete [ObjectType] "[Name]"

Error: [error message from tool]
```

---

### Known Limitations

**getObjectDependents cannot detect:**

1. **Field-level dependencies**
   - Fields are referenced by name in expressions: `recordType!RT.fields.fieldName`
   - Tool tracks object-level dependencies by UUID only
   - Use structural checks (relationships, views, title expression) + manual verification

2. **Parameter-level dependencies**
   - Cannot determine which parameters are passed to rule calls
   - Tool shows "rule X calls rule Y" but not which parameters are used
   - Required for "remove rule input" operations → use manual fallback

3. **Hard-coded string references**
   - Group names in text: `"PMS Administrators"`
   - Status values: `"Open"`, `"Closed"`
   - These are data values, not design object references

4. **Integration expression properties**
   - Integrations appear as dependents of connected systems (OUTBOUND_INTEGRATION type)
   - But expressions inside integration properties may not be tracked
   - Not yet validated - Phase 1B testing required

5. **Document/folder expression references**
   - Documents and folders may be referenced in expressions
   - Not yet validated as tracked dependency types

**For these limitations, use:**
- Manual verification fallback (if getObjectDependents fails)
- Structural checks (when applicable)
- record-types.md: Field deletion specific checks

---

### Example Final Outputs

These show what users see after you complete internal checks. Do NOT show "Step 5:" or tool call details to users.

#### Example 1: Record Type Delete with Expression + Structural Dependencies

**Operation:** Delete "PMS Status" record type

**Your internal process (NOT shown to user):**
```
1. Receive request: "Delete PMS Status record type"
2. Verify: getRecordType(uuid) → exists
3. Extract: name, type, app context
4. Identify: Delete record type → Expression + Structural
5. Check expression deps: getObjectDependents(uuid) → 47 dependents
6. Deduplicate → 5 unique interfaces, 2 unique rules, 1 unique PM
7. Check structural: getRecordType(uuid) → 2 relationships, listRecordTypeViews → 1 view, listRecordData → 4 records
8. Present using Record Type Delete Special Case template
```

**What user sees:**
```
⚠️ Cannot delete PMS Status: dependencies found

Expression dependencies (automated check):
❌ Interfaces: 5 unique (12 references) - will get "Unknown record type" errors
   - Interface Definition: Lines 15, 22, 34
   - Interface Definition: Lines 8, 45
   - Interface Definition: Lines 12, 13, 14
   ...and 2 more

❌ Expression Rules: 2 unique (3 references) - will fail validation
   - Expression Rule Definition: Line 10
   - Expression Rule Definition: Lines 5, 18

❌ Process Models: 1 unique (2 references) - nodes will fail
   - Process Variable > "status" > Data Type
   - Node > "Write Records" > Condition > Line: 1

Structural dependencies (confirmed):
❌ 2 record types reference Status:
   - PMS Project (relationship: status)
   - PMS Task (relationship: status)
❌ 1 custom view: Status Grid
❌ 4 status records exist (Created, Assigned, Approved, Closed)

Impact:
- 5 interfaces will break with "Unknown record type" errors
- 2 expression rules will fail validation
- 1 process model will have broken nodes/variables
- Project and Task relationships will break
- Status Grid view will fail to load
- 4 status records will be deleted

Options:
1. Cancel (recommended - remove dependencies first)
2. Proceed (HIGH RISK - breaks 8+ design objects)
3. Details (show full dependency list)

What would you like to do? (1/2/3)
```

---

#### Example 2: Constant Delete with 1 Dependency

**Operation:** Delete constant "CM_CASE_CONST"

**Your internal process (NOT shown to user):**
```
1. Receive request: "Delete constant CM_CASE_CONST"
2. Verify: getConstant(uuid) → exists
3. Extract: name (CM_CASE_CONST), type (RECORD_TYPE), value (uuid)
4. Identify: Delete constant → Expression deps
5. Check expression deps: getObjectDependents(uuid) → 2 dependents (1 APPLICATION, 1 FREEFORM_RULE)
6. Deduplicate → filter out APPLICATION → 1 unique rule
7. Present using Constant Delete Special Case template
```

**What user sees:**
```
⚠️ CRITICAL: You are about to DELETE constant "CM_CASE_CONST"

Constant details:
- Type: RECORD_TYPE
- Value: da7acb35-ea96-4cf7-accc-d06495ff21ac
- UUID: _a-0000f003-31cb-8000-9baa-011c48011c48_8516

Design object dependencies (automated check):
❌ Expression Rules: 1
   - Expression Rule Definition: Line 2

What breaks if you proceed:
❌ 1 expression rule will fail validation
❌ All cons!CM_CASE_CONST references will become "Unknown constant" errors

What's preserved:
✅ The referenced record type itself (UUID da7acb35...) is NOT deleted
✅ Other constants are not affected

Impact: Deleting will break 1 object.

Options:
1. Cancel (recommended - update 1 dependency first)
2. Proceed (you fix 1 broken reference)

Alternative approach (RECOMMENDED):
- Instead of deleting, consider if this constant is still needed
- If migrating to a different pattern, update the constant's value first

Type 'DELETE CM_CASE_CONST' to confirm HIGH RISK operation.
```

---

<a name="record-type-delete-special-case"></a>
### Record Type Delete Special Case

**PRESENTATION TEMPLATE - Used by Universal Workflow 1 Step 7**

This is a template, not a workflow. Universal Workflow 1 handles:
- ✅ Calling `getObjectDependents` (Step 5)
- ✅ Performing structural checks (Step 6)
- ✅ Deduplicating and grouping results (Step 5)
- ✅ Choosing this template (Step 7)

Your job: Present dependency results using the format below.

---

**Important:** `deleteRecordType` removes the **Appian metadata** (record type definition, relationships, views, actions) but does NOT drop the **database table**.

---

```
⚠️ You are about to DELETE record type "[Name]" (UUID: [uuid])

Expression dependencies (automated check via getObjectDependents):
[ONLY show object types that were actually returned. If zero of a type, skip it entirely.]

[If any interfaces found:]
❌ Interfaces: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])
   [Show all if < 10, otherwise show first 10 + "...and N more"]

[If any expression rules found:]
❌ Expression Rules: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])

[If any process models found:]
❌ Process Models: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])

[If any Web APIs found:]
❌ Web APIs: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])

[If zero expression dependencies found:]
✅ No design objects reference this record type (checked automatically)

Structural dependencies (from Step 6 structural checks):
[If any relationships:] ❌ N relationships to other record types: [list names]
[If any views:] ❌ N record views: [list names]
[If any actions:] ❌ N record actions: [list names]
[If any data:] ❌ N existing data records
[If zero structural dependencies:] ✅ No relationships, views, actions, or data records

Database impact:
⚠️ The database table [TABLE_NAME] will be PRESERVED
[If data exists:] ⚠️ N data records remain in database but will no longer be accessible via Appian
⚠️ To drop the table, manual database administration is required

Impact:
[Build this dynamically based on what was actually found - don't list types with zero dependencies]
[If interfaces found:] - N interfaces will break with "Unknown record type" errors
[If expression rules found:] - N expression rules will fail validation
[If process models found:] - N process models will have broken nodes/variables
[If relationships found:] - N relationships will break
[If views found:] - N views will fail to load
[If data found:] - N records become inaccessible (table preserved)

Options:
1. Cancel (recommended - remove dependencies first)
2. Proceed (HIGH RISK - breaks [TOTAL] design objects)
[If 10+ total dependencies:] 3. Details (show full dependency list)

Type 'DELETE [Name]' to confirm HIGH RISK operation.
```

**IMPORTANT:** 
- Do NOT add "Manual verification required" text
- Do NOT list object types that getObjectDependents didn't return
- If getObjectDependents found 0 expression dependencies, say "✅ No design objects reference this record type"
- Still show structural dependencies (relationships/views/actions/data) - these come from Step 6

**Why this matters:**
- User expectations: clarifies that data is preserved
- Prevents confusion when database table still exists
- Explains two-layer architecture (Appian metadata vs database)

---

<a name="constant-delete-special-case"></a>
### Constant Delete Special Case

**PRESENTATION TEMPLATE - Used by Universal Workflow 1 Step 7**

This is a template, not a workflow. Universal Workflow 1 handles:
- ✅ Calling `getObjectDependents` (Step 5)
- ✅ Deduplicating and grouping results (Step 5)
- ✅ Choosing this template (Step 7)

Your job: Present dependency results using the format below.

---

**Important:** Constants are referenced by **NAME** in expressions (`cons!CONSTANT_NAME`), not by UUID. Deleting a constant breaks **ALL** expressions that reference it, with **no automatic updates**.

---

**If dependencies found:**
```
⚠️ CRITICAL: You are about to DELETE constant "[Name]"

Constant details:
- Type: [TYPE] (GROUP, TEXT, INTEGER, RECORD_TYPE, etc.)
- Value: "[value]"
- Description: "[description]"
- UUID: [uuid]

Design object dependencies (automated check via getObjectDependents):
[ONLY show object types that were actually returned. If zero of a type, skip it entirely.]

❌ Expression Rules: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])
   [Show all if < 10, otherwise show first 10 + "...and N more"]
   
[If any interfaces found:]
❌ Interfaces: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])

[If any Web APIs found:]
❌ Web APIs: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])

[If any process models found:]
❌ Process Models: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])

What breaks if you proceed:
[Only list types that were actually found - don't mention types with zero dependencies]
❌ [N] expression rules will fail validation (cons![Name] becomes "Unknown constant")
[If interfaces found:] ❌ [N] interfaces will get "Unknown constant" errors
[If Web APIs found:] ❌ [N] Web APIs will fail validation
[If process models found:] ❌ [N] process models will have broken expressions

What's preserved:
✅ The referenced object itself (e.g., the group "[value]" is not deleted)
✅ Other constants are not affected

Impact: Deleting will break [TOTAL] objects. All cons![Name] references will become "Unknown constant" errors.

Options:
1. Cancel (recommended - update [TOTAL] dependencies first)
2. Proceed (you fix [TOTAL] broken references)
[If 10+ total dependencies:] 3. Details (show full dependency list)

Alternative approach (RECOMMENDED):
- Instead of deleting, update the constant's value to reference a different object
- This avoids breaking everything at once
- Allows gradual migration if needed

Type 'DELETE [Name]' to confirm HIGH RISK operation.
```

**IMPORTANT:** 
- Do NOT add "Manual verification required" text
- Do NOT suggest checking for types that getObjectDependents didn't return
- If getObjectDependents found 1 expression rule and 0 of everything else, ONLY show expression rules section
- Trust the tool - it checks ALL design objects automatically

**If NO dependencies found:**
```
✅ Safe to delete constant "[Name]"

Constant details:
- Type: [TYPE]
- Value: "[value]"
- UUID: [uuid]

Dependency check (automated):
✅ No design objects reference this constant

What happens:
- Constant "[Name]" will be permanently deleted
- No expressions will break (zero references found)
- The referenced object itself ("[value]") is preserved

Options:
1. Delete constant
2. Cancel

Proceed with delete? (yes/no)
```

**Why this matters:**
- Constants are name-based references (unlike UUID-based record types)
- No automatic update mechanism when constant is deleted
- Expression failures can be silent (no compile-time checks)
- Security implications if constant controls access

---

### Expression Rule Delete Special Case

**PRESENTATION TEMPLATE - Used by Universal Workflow 1 Step 7**

This is a template, not a workflow. Universal Workflow 1 handles:
- ✅ Calling `getObjectDependents` (Step 5)
- ✅ Deduplicating and grouping results (Step 5)
- ✅ Choosing this template (Step 7)

Your job: Present dependency results using the format below.

---

**Important:** Expression rules are referenced by **NAME** in expressions (`rule!RULE_NAME`), not by UUID. Deleting an expression rule breaks **ALL** expressions that call it, with **no automatic updates**.

---

**If dependencies found:**
```
⚠️ CRITICAL: You are about to DELETE expression rule "[RuleName]"

Rule details:
- Inputs: [input1, input2, ...]
- Description: "[description]"
- UUID: [uuid]

Design object dependencies (automated check via getObjectDependents):
[ONLY show object types that were actually returned. If zero of a type, skip it entirely.]

[If any calling expression rules found:]
❌ Expression Rules: N unique (M references) - calling rules
   - [Object name] (Line X: [breadcrumb details])
   [Show all if < 10, otherwise show first 10 + "...and N more"]

[If any interfaces found:]
❌ Interfaces: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])

[If any Web APIs found:]
❌ Web APIs: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])

[If any process models found:]
❌ Process Models: N unique (M references)
   - [Object name] (Line X: [breadcrumb details])

What breaks if you proceed:
[Only list types that were actually found - don't mention types with zero dependencies]
[If expression rules found:] ❌ [N] expression rules will fail validation (broken rule! references)
[If interfaces found:] ❌ [N] interfaces will get "Unknown rule" errors at runtime
[If Web APIs found:] ❌ [N] Web APIs will fail validation
[If process models found:] ❌ [N] process models will have broken expressions

Impact: Deleting will break [TOTAL] objects. All rule![RuleName] references will become "Unknown rule" errors.

Options:
1. Cancel (recommended - update [TOTAL] dependencies first)
2. Proceed (you fix [TOTAL] broken references)
[If 10+ total dependencies:] 3. Details (show full dependency list)

Alternative approach (RECOMMENDED):
- Instead of deleting, deprecate and create replacement rule
- Add "DEPRECATED" to rule name/description
- Create new rule with better implementation
- Migrate callers gradually
- Delete old rule when migration complete

Type 'DELETE [RuleName]' to confirm HIGH RISK operation.
```

**IMPORTANT:** 
- Do NOT add "Manual verification required" text
- Do NOT suggest checking for types that getObjectDependents didn't return
- Trust the tool - it checks ALL design objects automatically

**If NO dependencies found:**
```
✅ Safe to delete expression rule "[RuleName]"

Rule details:
- Inputs: [input1, input2, ...]
- UUID: [uuid]

Dependency check (automated):
✅ No design objects call this rule

What happens:
- Expression rule "[RuleName]" will be permanently deleted
- No expressions will break (zero references found)

Options:
1. Delete expression rule
2. Cancel

Proceed with delete? (yes/no)
```

**Why this matters:**
- Expression rules are name-based references
- No automatic update mechanism when rule is deleted
- Cascading failures through calling rules
- Runtime errors (no compile-time checks)

---

<a name="expression-rule-breaking-changes-special-case"></a>
### Expression Rule Breaking Changes Special Case

**PRESENTATION TEMPLATE - Used by Universal Workflow 1 Step 7**

This is a template, not a workflow. Universal Workflow 1 handles:
- ✅ Calling `getObjectDependents` (Step 5)
- ✅ Deduplicating and grouping results (Step 5)
- ✅ Choosing this template (Step 7)

Your job: Present dependency results using the format below.

---

**Important:** Expression rule breaking changes (input removal, input type change, return type change) break **ALL** calling rules, interfaces, and process models with **no automatic updates**.

---

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

Design object dependencies (automated check):
❌ Expression Rules: N unique (M references) - ALL will break
   [Show top 10 calling rules with breadcrumbs]
   
❌ Interfaces: N unique (M references) - ALL will error
   [Show top 10 with breadcrumbs]
   
❌ Web APIs: N unique (M references) - ALL will fail
   [Show all or top 10 with breadcrumbs]
   
❌ Process Models: N unique (M references) - ALL will break
   [Show all or top 10 with breadcrumbs]

What breaks if you proceed:
❌ [TOTAL] design objects will break when calling rule![RuleName]([inputName]: ...)
❌ All callers must be manually updated to remove [inputName] parameter
❌ No automatic migration available

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


<a name="note-on-renames"></a>
## Note on Renames

**Renaming Appian objects is safe and requires no dependency checking.**

All Appian objects use internal identifiers (UUIDs or integer IDs), not names. When you rename an object, only the display name changes - internal references continue working automatically.

### How Renames Work

| Object Type | Internal Reference | Rename Impact |
|---|---|---|
| Expression Rules | UUID | ✅ `rule!NAME()` syntax auto-updates to new name |
| Groups | Integer ID | ✅ All references continue working (ID unchanged) |
| Record Types | UUID | ✅ `recordType!` references continue working |
| Interfaces | UUID | ✅ All references continue working |
| Process Models | UUID | ✅ All references continue working |
| All other objects | UUID | ✅ All references continue working |
| **Constants** | **N/A** | **❌ Cannot be renamed (blocked by Appian UI)** |

### Rename Workflow

Simple process, no dependency checking needed:

1. **Verify object exists**
   - Call `get` or `list` operation to confirm object exists
   - Resolve name to UUID if needed

2. **Check if rename is allowed**
   - Constants: Cannot be renamed (Appian platform limitation)
   - All other objects: Can be renamed safely

3. **Present simple confirmation**
   ```
   Rename "[ObjectType]" from "[OldName]" to "[NewName]"?
   
   ✅ Safe operation - all references will continue working
   (Internal identifiers remain unchanged)
   
   Continue? (yes/no)
   ```

4. **Execute rename**
   - Call appropriate `update` tool with new name
   - UUID/ID remains unchanged

5. **Inform user**
   ```
   ✅ Renamed "[ObjectType]" from "[OldName]" to "[NewName]"
   
   All references automatically updated.
   ```

### Special Case: Constants Cannot Be Renamed

If user requests constant rename:

```
❌ Cannot rename constants

Appian does not support renaming constants through the UI or API.

Alternative approach:
1. Create new constant with new name and same value
2. Update expressions to use new constant (search for cons!OLD_NAME)
3. Delete old constant when all references are updated

This gradual migration avoids breaking all expressions at once.
```

### Why This Is Safe

**Technical explanation:**

- **Expression rules:** Even though syntax is `rule!NAME()`, Appian stores the UUID internally. When you rename, Appian updates the syntax everywhere.
- **Groups:** Functions like `a!isUserMemberOfGroup()` accept Group objects (which have integer IDs). Constants store the Group object, not the name string.
- **Record types, interfaces, etc.:** All use UUID-based references internally.

**The only name-based reference in Appian is `cons!NAME`, and constants cannot be renamed.**

---


<a name="universal-workflow-7-dependency-checking"></a>
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
