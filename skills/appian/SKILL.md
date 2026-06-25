---
name: "appian"
description: "MANDATORY skill for Appian MCP tool usage. Provides critical domain knowledge (naming conventions, relationship rules, data modeling patterns, dependency order, UUID handling) that MCP tool schemas cannot express. Load this skill BEFORE calling any Appian MCP tools. Covers: record types, interfaces, expression rules, process models, sites, Web APIs, data modeling, relationships, SAIL expressions, security, change planning."
---

## CRITICAL: Read This Before Using Appian MCP Tools

**Stop.** If you are about to call Appian MCP tools (`createRecordType`, `addRecordTypeRelationship`, `createInterface`, etc.), you MUST load reference files from this skill first.

**Why this matters:**

MCP tool schemas describe **parameters** (what fields exist), but not **domain requirements** (how to use them correctly):

- ❌ Tool schema: "`createRecordType` accepts `name`, `fields`, `sourceType`"
- ✅ Skill reference: "Primary key must be named `id` (INTEGER), USER fields require SYSTEM_RECORD_TYPE_USER relationships, relationships need both MANY_TO_ONE + ONE_TO_MANY declarations"

**Without this skill, you will:**
- Create broken relationships (missing reverse sides)
- Use wrong naming conventions (404 errors, convention violations)
- Omit mandatory relationships (USER fields won't display correctly)
- Create objects in wrong order (dependency failures)
- Fabricate UUIDs (silent data corruption)

**Before calling ANY Appian MCP tool, follow the loading strategy below.**

---

## Configuration

**Appian Version:** 26.5

**Supported Versions (as of 2026-06-25):**
26.5, 26.3, 25.4, 25.3, 25.2, 25.1, 24.4, 24.3

Update this version to match your Appian environment. This affects:
- Documentation URL lookups
- Function availability checks
- Version-specific guidance

**To change:** Edit the version number above.

**Maintenance:** Update this list quarterly when new Appian versions are released. Typically add the latest version and remove the oldest (2+ years old).

**Last updated:** 2026-06-25

---

## Tool Surface

Appian MCP tools have names like `createApplication`, `createRecordType`, `addRecordTypeRelationship`, `listInterfaces`, `getProcessModel`. If you see these in your tool list (regardless of prefix), this skill is MANDATORY.

Tool schemas are self-describing for parameter structure. Load `references/tools-mcp.md` for usage patterns, UUID handling, and non-obvious behaviors the schemas don't communicate.

---

## Resource Reference Map

Each resource has a dedicated reference file with JSON schemas, design conventions, and pitfalls **that tool schemas cannot express**. 

**Loading is NOT optional** — reference files contain mandatory requirements (naming rules, relationship patterns, dependency constraints) that will cause failures if ignored.

Load the relevant reference(s) for your task:

| When to load | Reference File |
|---|---|
| You need usage patterns for the Appian MCP tools | `references/tools-mcp.md` |
| Understanding when to ask user for confirmation vs auto-complete mandatory steps | `references/confirmation-patterns.md` |
| Writing any expression or expression rule (ALWAYS load core pattern files) | `references/function-reference.md`, `references/null-safety-patterns.md`, `references/short-circuit-patterns.md` |
| Using Appian functions, operators, or type conversions in expressions | `references/function-reference.md`, `references/null-safety-patterns.md` |
| Need detailed array, date/time, match, or forEach patterns | `references/function-patterns-index.md` (loads pattern files on demand) |
| Querying record types with filters, sorting, relationships, aggregations (KPIs), or paging | `references/query-record-type-patterns.md` |
| Creating or managing an application | `references/applications.md` |
| Creating/modifying record types, fields, relationships, views, or actions | `references/record-types.md` |
| Requirements mention filtering, searching, faceted navigation, or record list dropdowns | `references/record-type-user-filters.md` |
| Creating/modifying interfaces or writing SAIL form expressions | `references/interfaces.md` |
| Creating/modifying expression rules (architectural guidance) | `references/expression-rules.md` |
| Gathering expression rule requirements (clarifying inputs, outputs, validation depth, query scope before implementation) | `references/expressions.md` |
| Managing expression rules (create vs update vs version) | `references/expressions.md` |
| Expression rule architectural decisions (when to inline vs extract, performance pitfalls) | `references/expressions.md` |
| Creating/modifying process models, adding nodes, or wiring start forms | `references/process-models.md` |
| Creating/modifying sites or adding pages | `references/sites.md` |
| Creating constants, groups, folders, or documents | `references/supporting-objects.md` |
| Designing a data model, choosing entity structure, or normalizing fields into lookup tables | `references/data-modeling.md` |
| Writing SAIL expressions for interfaces (layout, components, patterns) | `references/sail.md` |
| Configuring security roles, record-level security, or group hierarchy | `references/security.md` |
| Configuring security expressions, group hierarchies, or role-based access patterns | `references/security-patterns.md` |
| Starting a multi-object task — need to plan dependency order and scope | `references/change-planning.md` |
| Validating or testing completed changes | `references/change-review.md` |
| Choosing field types or need type constraints (length, precision) | `references/field-types.md` |
| Configuring record events, writing events in process models, displaying event history in interfaces, or enabling process mining (Process HQ). Requirements mention: auditing, activity log, event history, tracking changes, collaboration on records, process mining. | `references/record-events.md` |
| Building a dashboard, form layout, or summary view | `references/ui-patterns.md` |
| Need to look up a specific SAIL component's parameters | `references/component-reference.md` |

### MANDATORY Loading Strategy

**Before calling any Appian MCP tools, load reference files in this order:**

#### Step 1: ALWAYS Load Universal Patterns First
```
Load: references/tools-mcp.md
Load: references/confirmation-patterns.md
Load: references/function-reference.md
Load: references/null-safety-patterns.md
Load: references/short-circuit-patterns.md
```

These cover universal patterns across all Appian tasks:
- `tools-mcp.md`: UUID handling, update behaviors, CSV formats, tool-specific conventions
- `confirmation-patterns.md`: When to ask user for confirmation vs auto-complete mandatory steps
- `function-reference.md`: Function catalog, anti-hallucination list, signatures
- `null-safety-patterns.md`: Null handling, functions that reject null, standard safety patterns
- `short-circuit-patterns.md`: Nested if() patterns for safe conditional evaluation

**Non-negotiable for all Appian work.**

**For detailed patterns (load on demand):**
- Need detailed array/date/match/forEach patterns → Load `references/function-patterns-index.md` for navigation

#### Step 2: Load Primary Domain Reference
Use the Resource Reference Map above to identify which reference file matches your task, then load it.

**Common scenarios:**
- Creating record types → Load `references/record-types.md` AND `references/data-modeling.md`
- Adding relationships → Load `references/relationship-patterns.md`
- Building interfaces → Load `references/interfaces.md` AND `references/sail.md`
- Creating process models → Load `references/process-models.md` AND `references/node-types.md`

#### Step 3: Load Supplementary References
Based on what you discover in Step 2, load additional references:
- Field type constraints → `references/field-types.md`
- Security configuration → `references/security.md`
- Multi-object tasks → `references/change-planning.md`

#### Step 4: Only Now Call MCP Tools
After loading references, you have the domain knowledge to call tools correctly.

**Validation checklist before calling tools:**
- [ ] Loaded all 5 required files from Step 1? (tools-mcp, confirmation-patterns, function-reference, null-safety-patterns, short-circuit-patterns)
- [ ] Checked anti-hallucination list before using functions? (regexmatch, property(), a!dateTimeValue() don't exist!)
- [ ] Loaded primary domain reference for this task?
- [ ] Understand naming conventions (table names, field names, relationship names)?
- [ ] Understand dependency order (what must exist before creating this object)?
- [ ] Have actual UUIDs from environment (not fabricated)?
- [ ] Know which relationships are mandatory (e.g., USER fields → SYSTEM_RECORD_TYPE_USER)?
- [ ] Need detailed patterns? Load `references/function-patterns-index.md` for navigation

---

## Dependency Order

Appian objects must be created in dependency order. Later objects reference earlier ones:

1. Application (creates default groups and folders)
2. Groups (additional role groups)
3. Folders (additional sub-folders)
4. Constants (reference groups, store config values)
5. Record types (with fields)
6. Record type relationships (requires all record types to exist)
7. Expression rules
8. Interfaces
9. Process models
10. Record type actions, views, filters (reference process models and interfaces)
11. Sites (reference interfaces)
12. Web APIs
13. Documents (uploaded to folders)

---

## Common Failure Modes (What Happens When You Skip Skills)

These are real errors that occur when MCP tools are called without loading reference files:

**❌ Skipping USER field relationships:**
- Symptom: USER fields display as plain text (no name, email, profile picture)
- Cause: Created USER field but didn't add MANY_TO_ONE relationship to SYSTEM_RECORD_TYPE_USER
- Prevention: Load `references/record-types.md` + `references/relationship-patterns.md` (documents mandatory USER field pattern)

**❌ Creating only one side of relationship:**
- Symptom: Can navigate Ticket → Status but not Status → Tickets in UI
- Cause: Created MANY_TO_ONE from Ticket to Status, but missing ONE_TO_MANY reverse relationship
- Prevention: Load `references/relationship-patterns.md` (documents bidirectional requirement)

**❌ Wrong naming conventions:**
- Symptom: 404 errors, convention violations, failed validations
- Cause: Used `statusName` instead of `name` for lookup tables, or `customerId` as primary key instead of `id`
- Prevention: Load `references/data-modeling.md` (documents naming rules)

**❌ Fabricating UUIDs:**
- Symptom: Silent failures, 404 Not Found, data corruption
- Cause: Guessed UUID format or reused UUIDs from different environments
- Prevention: Load `references/tools-mcp.md` (documents UUID sourcing rules)

**❌ Wrong dependency order:**
- Symptom: Creation fails because referenced object doesn't exist yet
- Cause: Created interface before creating the record type it references
- Prevention: Load `references/change-planning.md` (documents dependency sequence)

**❌ Creating application without group hierarchy:**
- Symptom: Flat group structure (all groups at same level), no permission inheritance, missing group constants for security expressions
- Cause: Called `createApplication` and `createGroup` without loading `security-patterns.md`
- Example: Created role groups (e.g., "PREFIX Managers", "PREFIX Workers", "PREFIX Reviewers") but didn't set parent-child relationships → manual permission management required on every object
- Prevention: Load `references/applications.md` → redirects to `references/security-patterns.md` for group hierarchy template

## Universal Tips

- Always discover what exists before creating (list before create)
- Get an object before updating it — updates replace provided fields entirely
- Store UUIDs in variables for multi-step workflows
- Record type relationships require both sides declared (MANY_TO_ONE + ONE_TO_MANY)
- All `recordType!` references in expressions must use UUID-qualified format: `'recordType!{uuid}Name.fields.{fieldUuid}fieldName'`

---

## Appian Documentation Search (For Undocumented Functions)

When you encounter a function not documented in function-reference.md, use this workflow to look it up in Appian's official documentation.

### Step 1: Read configured version

Read the **Appian Version** from the Configuration section above. Use this version for all documentation URLs.

### Step 2: Check functions.json for function existence

**First lookup in session (cache for reuse):**
```bash
# Read configured version from Configuration section
VERSION="26.5"  # Use the configured version

# Fetch and cache functions.json
curl -s "https://docs.appian.com/suite/help/$VERSION/functions.json" \
  > /tmp/appian-functions-$VERSION.json
```

**Subsequent lookups (use cached):**
```bash
# Check if cached first
if [ ! -f /tmp/appian-functions-$VERSION.json ]; then
  curl -s "https://docs.appian.com/suite/help/$VERSION/functions.json" \
    > /tmp/appian-functions-$VERSION.json
fi

# Look up function (case-insensitive)
jq -r '.["a!queryrecordtype"]' /tmp/appian-functions-$VERSION.json
# Returns: "/suite/help/26.5/fnc_system_queryrecordtype.html" if exists
# Returns: null if doesn't exist
```

### Step 3: Fetch documentation page

```bash
# If function exists, fetch full documentation
DOC_PATH=$(jq -r '.["a!queryrecordtype"]' /tmp/appian-functions-$VERSION.json)

if [ "$DOC_PATH" != "null" ]; then
  curl -s "https://docs.appian.com$DOC_PATH"
fi
```

### Step 4: Extract key information

From the documentation page, extract:
- Function name and signature
- Parameters (name, type, required/optional, description)
- Return type
- Usage notes and examples
- Related functions

### When to use this workflow

**Use for:**
- ✅ Expression validation error mentions unknown function
- ✅ User asks about a function not in function-reference.md
- ✅ Need to verify function signature or parameters
- ✅ Function exists but signature unclear

**Don't use for:**
- ❌ Functions already in function-reference.md (trust our curated docs first)
- ❌ Every function (only when needed)
- ❌ Patterns/recipes (that's Phase 1b - different search mechanism)

### Fallback behavior

If configured version is not found or Configuration section is missing:
1. Default to version 26.5 (latest)
2. Suggest to user: "Using Appian 26.5 docs. To use a different version, update the Configuration section in skills/appian/SKILL.md"
