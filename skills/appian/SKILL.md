---
name: "appian"
description: "MANDATORY skill for Appian MCP tool usage. Provides critical domain knowledge (naming conventions, relationship rules, data modeling patterns, dependency order, UUID handling) that MCP tool schemas cannot express. Load this skill BEFORE calling any Appian MCP tools. Covers: record types, interfaces, expression rules, process models, sites, Web APIs, data modeling, relationships, SAIL expressions, security, accessibility auditing, change planning."
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

**Appian Version:** 26.6

**Supported Versions (as of 2026-06-30):**
26.6, 26.5 (redirects to 26.6), 26.3, 25.4, 25.3, 25.2, 25.1, 24.4

Update this version to match your Appian environment. This affects:
- Documentation URL lookups
- Function availability checks
- Version-specific guidance

**To change:** Edit the version number above.

**Version Notes:**
- Most versions have dedicated documentation (26.6, 26.3, 25.4, 25.3, etc.)
- Version 26.5 redirects to 26.6 documentation (no separate 26.5 files)
- If your environment is 26.5, use 26.6 configuration (documentation is compatible)
- The VERSION variable pattern works for all supported versions

**Maintenance:** Update this list quarterly when new Appian versions are released. Typically add the latest version and remove the oldest (2+ years old).

**Last updated:** 2026-06-30

---

## Tool Surface

Appian MCP tools have names like `createApplication`, `createRecordType`, `addRecordTypeRelationship`, `listInterfaces`, `getProcessModel`. If you see these in your tool list (regardless of prefix), this skill is MANDATORY.

Tool schemas are self-describing for parameter structure. Load `references/tools-mcp.md` for usage patterns, UUID handling, and non-obvious behaviors the schemas don't communicate.

---

## ⚠️ CRITICAL: Deletion Operations Workflow

**STOP: Before deleting ANY Appian object, you MUST follow this workflow.**

This applies to ALL deletion operations:
- Delete constant, expression rule, record type, interface, process model, Web API, site, connected system
- Delete field, relationship, view, action (record type components)
- Delete group, application, folder, document

### Mandatory Pre-Deletion Steps

**1. ALWAYS Load `references/confirmation-patterns.md` FIRST**

This file contains **Universal Workflow 1: Delete Confirmation** — the complete workflow for safe deletions.

```
Load: references/confirmation-patterns.md
```

**Why this is mandatory:**
- Prevents breaking production applications
- Automatically detects what will break before deletion
- Provides user-facing presentation templates
- Shows impact assessment with line-level precision

**2. Follow Universal Workflow 1 Exactly — ALL 10 Steps, NO EXCEPTIONS**

After loading confirmation-patterns.md, follow all 10 steps IN ORDER:

- **Steps 1-3:** Receive request, verify object, extract details
- **Step 4:** Identify operation type (determines which dependency checks apply)
- **Step 5:** Check expression dependencies using `getObjectDependents` tool ← MANDATORY
- **Step 6:** Perform structural checks (relationships, views, data, hierarchy)
- **Step 7:** Present dependencies to user (use templates — NOT your own format)
- **Step 8:** Offer resolution strategies
- **Step 9:** Get user confirmation ← MANDATORY, NEVER SKIP THIS
- **Step 10:** Execute deletion

**⚠️ CRITICAL: Step 9 is MANDATORY even if zero dependencies found.**
- You MUST present findings to user
- You MUST wait for explicit confirmation
- You MUST NOT delete until user confirms
- Deletion is permanent and irreversible — user deserves final say

**3. Use the Templates in Step 4**

The templates in confirmation-patterns.md show EXACTLY what users should see:
- ✅ Clean dependency lists grouped by type
- ✅ Impact statements ("N objects will break")
- ✅ Action options (Cancel / Proceed / Details)
- ❌ NO tool call details ("getObjectDependents(uuid) →")
- ❌ NO step numbers ("Step 2: Expression dependencies")
- ❌ NO processing details ("Deduplicating by UUID...")

### Why This Matters

**Without this workflow:**
- You might delete objects with active dependencies
- User sees broken "Unknown constant" or "Unknown record type" errors
- Production applications fail at runtime
- No clear impact assessment before deletion

**With this workflow:**
- User sees exactly what will break BEFORE confirming deletion
- Automated dependency detection with line-level precision
- Clear presentation with structured options
- Follows Appian best practices for safe deletions

### Quick Reference: Common Deletions

| Operation | Files to Load | Workflow Steps |
|-----------|---------------|----------------|
| Delete constant | confirmation-patterns.md + supporting-objects.md | Steps 1-4 → Step 5 (getObjectDependents) → Step 7 (present) → Step 9 (confirm) → Step 10 (execute) |
| Delete expression rule | confirmation-patterns.md + expression-rules.md | Steps 1-4 → Step 5 (getObjectDependents) → Step 7 (present) → Step 9 (confirm) → Step 10 (execute) |
| Delete record type | confirmation-patterns.md + record-types.md | Steps 1-4 → Step 5 (getObjectDependents) + Step 6 (structural) → Step 7 (present) → Step 9 (confirm) → Step 10 (execute) |
| Delete interface | confirmation-patterns.md + interfaces.md | Steps 1-4 → Step 5 (getObjectDependents) → Step 7 (present) → Step 9 (confirm) → Step 10 (execute) |
| Delete group | confirmation-patterns.md + supporting-objects.md | Steps 1-4 → Step 6 (structural: hierarchy, members) → Step 7 (present) → Step 9 (confirm) → Step 10 (execute) |

**Non-negotiable:** Load confirmation-patterns.md for ALL deletion operations, no exceptions.

**Note:** Universal Workflow 1 is now the complete deletion workflow (10 steps). Previous "Workflow 7" has been merged into Workflow 1.

---

## Resource Reference Map

Each resource has a dedicated reference file with JSON schemas, design conventions, and pitfalls **that tool schemas cannot express**. 

**Loading is NOT optional** — reference files contain mandatory requirements (naming rules, relationship patterns, dependency constraints) that will cause failures if ignored.

Load the relevant reference(s) for your task:

| When to load | Reference File |
|---|---|
| You need usage patterns for the Appian MCP tools | `references/tools-mcp.md` |
| **Deleting or updating ANY Appian object (MANDATORY - see Deletion Operations section above)** | `references/confirmation-patterns.md` |
| Validating SAIL expressions before creating interfaces or expression rules (Step 7B - MANDATORY for all interface/rule creation) | `references/validation-checkpoint.md` |
| Writing any expression or expression rule (ALWAYS load core pattern files) | `references/function-reference.md`, `references/null-safety-patterns.md`, `references/short-circuit-patterns.md` |
| Using Appian functions, operators, or type conversions in expressions | `references/function-reference.md`, `references/null-safety-patterns.md` |
| Need detailed array, date/time, match, or forEach patterns | `references/function-patterns-index.md` (loads pattern files on demand) |
| Need three-tier documentation lookup workflow (functions.json, doc search) | `references/documentation-lookup-strategy.md` |
| Verifying SAIL component existence or looking up component instruction files | `registry/components-registry.json` (comprehensive component list with existence flags and instruction file mapping) |
| Querying record types with filters, sorting, relationships, aggregations (KPIs), or paging | `references/query-record-type-patterns.md` |
| Using date/time filtering, arithmetic, or comparisons in queries or expressions. Requirements mention: dates, times, today(), now(), date ranges, past/future dates, intervals | `references/date-time-patterns.md` |
| Creating or managing an application | `references/applications.md` |
| Naming conventions for objects | Load reference files relevant to the object type, such as: `references/interfaces.md`, `references/expression-rules.md`, `references/supporting-objects.md`, `references/sites.md`, `references/process-models.md`, `references/record-types.md`, or `references/data-modeling.md` (NOTE: some object types do not have naming conventions) |
| Creating/modifying record types, fields, relationships, views, or actions | `references/record-types.md` |
| Requirements mention filtering, searching, faceted navigation, or record list dropdowns | `references/record-type-user-filters.md` |
| Understanding interface types, workflows (Dashboard → Summary View → Form), and when to use each type (MANDATORY before creating any interface) | `references/appian-workflow-patterns.md` |
| Creating record summary views (read-only detail pages for primary entities showing related records) | `references/record-summary-views.md` |
| Creating/modifying interfaces or writing SAIL form expressions | `references/interfaces.md` |
| Building dropdown fields with Priority, Status, Type, Category lookups. Requirements mention: dropdown, select, priority, status, lookup table | `references/dropdown-patterns.md` |
| Creating/modifying expression rules (architectural guidance) | `references/expression-rules.md` |
| Gathering expression rule requirements (clarifying inputs, outputs, validation depth, query scope before implementation) | `references/expressions.md` |
| Managing expression rules (create vs update vs version) | `references/expressions.md` |
| Expression rule architectural decisions (when to inline vs extract, performance pitfalls) | `references/expressions.md` |
| Creating/modifying process models, adding nodes, or wiring start forms | `references/process-models.md` |
| Creating/modifying sites or adding pages | `references/sites.md` |
| Creating constants, groups, folders, or documents | `references/supporting-objects.md` |
| Designing a data model, choosing entity structure, or normalizing fields into lookup tables | `references/data-modeling.md` |
| Writing SAIL expressions for interfaces (layout, components, patterns) | `references/sail.md` |
| Building editable forms with Save buttons (inline writes vs process-based updates) | `references/write-records-patterns.md` |
| Configuring security roles, record-level security, or group hierarchy | `references/security.md` |
| Configuring security expressions, group hierarchies, or role-based access patterns | `references/security-patterns.md` |
| Starting a multi-object task — need to plan dependency order and scope | `references/change-planning.md` |
| Validating or testing completed changes | `references/change-review.md` |
| Choosing field types or need type constraints (length, precision) | `references/field-types.md` |
| Configuring record events, writing events in process models, displaying event history in interfaces, or enabling process mining (Process HQ). Requirements mention: auditing, activity log, event history, tracking changes, collaboration on records, process mining. | `references/record-events.md` |
| Building a dashboard, form layout, or summary view | `references/ui-patterns.md` |
| Dashboard with KPI cards, list cards, message banners, or tabs | Load architectural pattern from `ui-patterns.md` PLUS component examples from `patterns/` directory (see Step 3 for which pattern files to load) |
| Creating any interface (final validation before writing SAIL code) | `references/interface-generation-checklist.md` (lightweight 10-point checklist - loaded at Step 6) |
| Need to look up a specific SAIL component's parameters | `references/component-reference.md` |
| Auditing interfaces for accessibility, fixing accessibility defects, or building accessible interfaces (WCAG compliance) | `references/accessibility-audit.md`, `references/component-checks.md`, `references/accessibility-reference.md` |

### MANDATORY Loading Strategy

**Before calling any Appian MCP tools, load reference files in this order:**

#### Step 1: ALWAYS Load Universal Patterns First
```
Load: references/tools-mcp.md
Load: references/confirmation-patterns.md
Load: references/function-reference.md
Load: references/component-reference.md
Load: references/null-safety-patterns.md
Load: references/short-circuit-patterns.md
Load: references/sail-verification-checkpoint.md
```

These cover universal patterns across all Appian tasks:
- `tools-mcp.md`: UUID handling, update behaviors, CSV formats, tool-specific conventions (ALWAYS)
- `confirmation-patterns.md`: **MANDATORY for ALL delete/update operations** — Universal Workflow 7 (Dependency Checking), user confirmation workflows, when to auto-complete vs ask
- `function-reference.md`: Function catalog, anti-hallucination list, signatures for expression rules (ALWAYS)
- `component-reference.md`: SAIL component catalog, anti-hallucination list, signatures for interfaces (ALWAYS)
- `null-safety-patterns.md`: Null handling, functions that reject null, standard safety patterns (ALWAYS)
- `short-circuit-patterns.md`: Nested if() patterns for safe conditional evaluation (ALWAYS)
- `sail-verification-checkpoint.md`: **MANDATORY checkpoint for verifying function and component existence before code generation** (ALWAYS)

**Non-negotiable for all Appian work.**

**Why load both function-reference.md AND component-reference.md?**
- Expression rules need functions (logic)
- Interfaces need components (UX) AND functions (logic in saveInto, validation, data transformation)
- Loading both ensures you have complete coverage for any Appian development task
- Both contain anti-hallucination lists checked in sail-verification-checkpoint.md (Step 4)

**For detailed patterns (load on demand):**
- Need detailed array/date/match/forEach patterns → Load `references/function-patterns-index.md` for navigation

#### Step 2: Load Primary Domain Reference
Use the Resource Reference Map above to identify which reference file matches your task, then load it.

**Common scenarios:**
- Creating record types → Load `references/record-types.md` AND `references/data-modeling.md`
- Adding relationships → Load `references/relationship-patterns.md`
- **Building any interface** → Load `references/appian-workflow-patterns.md` (MANDATORY - provides interface type decision criteria) AND `references/interfaces.md` AND `references/sail.md`
- Building interfaces (editable with Save button / Forms) → Load `references/appian-workflow-patterns.md` AND `references/interfaces.md` AND `references/sail.md` AND `references/write-records-patterns.md`
- Building interfaces (read-only / Summary Views) → Load `references/appian-workflow-patterns.md` AND `references/interfaces.md` AND `references/sail.md`
- Creating process models → Load `references/process-models.md` AND `references/node-types.md`

---

#### Step 3: Load UI Pattern Examples (for interfaces)

**If building an interface, check which pattern files apply:**

- [ ] **KPI cards?** → Load `references/patterns/kpis.md`
  - Complete examples: Revenue KPI, User KPI, Task completion KPI
  - Shows stamp + richText + growth indicator pattern
  - Sample data structures and color schemes

- [ ] **List of items as cards?** → Load `references/patterns/card_lists.md`
  - Complete examples: User cards, Task cards, Product cards
  - Shows cardGroupLayout + a!forEach + sideBySideLayout pattern
  - Sample a!map data with all required fields

- [ ] **Message banners?** → Load `references/patterns/messages.md`
  - Complete examples: Info, Warning, Success, Error messages
  - Shows messageBanner patterns with icons and colors

- [ ] **Tab navigation?** → Load `references/patterns/tabs.md`
  - Complete examples: Tab bar with multiple tabs
  - Shows tab state management and content switching

**Pattern files provide:**
- ✅ Complete working SAIL code (copy-paste ready)
- ✅ Correct data structure (a!map with proper fields)
- ✅ Layout composition (sideBySideLayout, cardGroupLayout)
- ✅ Consistent styling (color schemes, spacing)

**Skip patterns if:**
- ❌ Request doesn't mention these specific components
- ❌ Building simple forms or single components
- ❌ Creating expression rules (not interfaces)

---

### Step 4: [MANDATORY CHECKPOINT] Verify Functions & Components

🛑 **STOP. Follow the SAIL Verification Checkpoint now.**

🚫 **BLOCKING REQUIREMENT:** You MUST output verification artifacts before proceeding to Step 5.

**Action:** Open and complete `references/sail-verification-checkpoint.md`

**Requirements:**
- **Expression rules:** Complete Step 4A only (verify functions)
- **Interfaces:** Complete BOTH Step 4A (verify functions) AND Step 4B (verify components)

**What this prevents:**
- ❌ Using non-existent functions (regexmatch, property, a!dateTimeValue)
- ❌ Using non-existent components (a!richTextEditor)
- ❌ Missing critical warnings (showSearchBox only works with recordType! data)
- ❌ Fabricating parameters or signatures

**The checkpoint document contains:**
- Complete verification workflows with copy-paste bash commands
- Anti-hallucination list checks
- Tier 2A/2B verification procedures (functions.json lookups)
- Instruction file loading guidance
- Exit checklists for both Step 4A and Step 4B

**⚠️ If you proceed to Step 5 without outputting verification artifacts, STOP and return to Step 4.**

---

**REQUIRED OUTPUT (Before Step 5):**

For expression rules, output:
```
=== STEP 4A VERIFICATION COMPLETE ===
Functions to verify: [list actual function names]
Anti-hallucination check: ✅ (checked lines X-Y of function-reference.md)
Functions on NO-EXIST list found: [count or "none"]
Tier 2A verification: ✅ [count] functions | ⚠️ curl failed - fallback: checked function-reference.md
Functions verified as existing: [count]/[total]
Non-existent functions removed: [count or "none"]
Ready to proceed: YES
```

For interfaces, output BOTH:
```
=== STEP 4A VERIFICATION COMPLETE ===
Functions to verify: [list actual function names]
Anti-hallucination check: ✅ (checked lines X-Y of function-reference.md)
Functions on NO-EXIST list found: [count or "none"]
Tier 2A verification: ✅ [count] functions | ⚠️ curl failed - fallback: checked function-reference.md
Functions verified as existing: [count]/[total]
Non-existent functions removed: [count or "none"]
Ready to proceed: YES

=== STEP 4B VERIFICATION COMPLETE ===
Components to verify: [list actual component names]
Registry check: ✅ (all exist=true)
Components with instruction files:
  - [component] → [file path] ✅ loaded
  - [component] → [file path] ✅ loaded
Critical warnings noted:
  1. [direct quote from instruction file]
  2. [direct quote from instruction file]
Tier 2B verification: ✅ [count] components | skipped (all have instruction files)
Non-existent components removed: [count or "none"]
Ready to proceed: YES
```

🔍 **Verification Evidence Required:**
- List the actual functions/components being verified (not just counts)
- Show which instruction files were loaded (file paths)
- Note any failures or fallbacks (curl errors, missing files)
- Record any items removed due to non-existence
- Quote critical warnings directly from loaded instruction files

❌ **Invalid completion examples:**
- Generic counts without showing the list
- "✅" without showing what was checked
- "All passed" without evidence
- Placeholders like [count] or [list] left unfilled in output

🚫 **BLOCKING GATE: Cannot proceed to Step 5 without outputting verification artifacts above.**

**Self-Check Before Proceeding:**
- [ ] Did I output "=== STEP 4A VERIFICATION COMPLETE ===" with actual function list?
- [ ] Did I output "=== STEP 4B VERIFICATION COMPLETE ===" with actual component list? (interfaces only)
- [ ] Did I list instruction files that were loaded with file paths?
- [ ] Did I quote at least one critical warning from loaded instruction files? (interfaces only)

**If any checkbox is ❌, return to Step 4 and complete verification.**

---

#### Step 5: Load Supplementary References

🔍 **Step 5 Entry Guard:**

Before loading supplementary references, confirm:
- [ ] Step 4A completion artifact was output in this conversation ✅
- [ ] Step 4B completion artifact was output in this conversation ✅ (interfaces only)
- [ ] Artifact includes actual function/component lists (not placeholders)
- [ ] At least one instruction file was loaded for interfaces (if components have them)

If any ❌, return to Step 4.
Based on what you discover in Steps 2-4, load additional references:
- Field type constraints → `references/field-types.md`
- Security configuration → `references/security.md`
- Multi-object tasks → `references/change-planning.md`

#### Step 6: Final Pre-Implementation Verification

**Complete these actions before writing code:**

**For Expression Rules:**
1. **Verify functions** (already completed in Step 4)
   - Confirmed: anti-hallucination list checked, unknown functions looked up via Tier 2A
2. **Apply null safety patterns**
   - Use patterns from null-safety-patterns.md
   - Guard all operations: `if(a!isNullOrEmpty(value), default, operation)`
3. **Verify type handling**
   - Know input types (from rule parameters) and output type (return value)
   - Use explicit casting where needed (tointeger(), todecimal(), totext())

**For Interfaces:**
1. **Load interface generation checklist** (Quick validation before writing SAIL)
   ```
   Load: references/interface-generation-checklist.md
   ```
   - 10-point checklist covering: reference loading, data source clarity, chart patterns, layout hierarchy, component parameters, null safety, function variables, anti-patterns
   - Decision trees for chart data approach, pie chart pattern, layout selection
   - Takes 5-10 minutes to complete
   - **Critical for catching errors before calling MCP tools**

2. **Complete checklist items:**
   - Reference loading complete (Steps 1-4)
   - Data source clarity (mockup vs record data)
   - Chart patterns verified (correct approach chosen)
   - Layout hierarchy valid (no orphans, correct widths)
   - Component parameters exist (no fabrications)
   - Null safety applied throughout
   - Function variables correct (fv!item, local!)
   - Query structure valid (if using record data)
   - Anti-patterns avoided (checked non-existent list)

3. **Confirm readiness:**
   - If ALL checklist items pass → Proceed to Step 7
   - If ANY item fails → Stop and resolve before proceeding

**For Both:**
- **References loaded:** All 7 universal patterns (Step 1) + primary domain reference (Step 2) + UI patterns if applicable (Step 3)
- **Domain knowledge ready:** Naming conventions, dependency order, mandatory relationships
- **UUIDs available:** From environment (via list/get operations), not fabricated

#### Step 7: Generate, Validate, and Create Objects

After completing Steps 1-6 verification, you have complete implementation knowledge. Now follow this three-phase workflow:

**Final pre-generation checklist:**
- [ ] Loaded all 7 required files from Step 1? (tools-mcp, confirmation-patterns, function-reference, component-reference, null-safety-patterns, short-circuit-patterns, sail-verification-checkpoint)
- [ ] Passed Step 4 checkpoint? (completed sail-verification-checkpoint.md exit checklist)
  - [ ] Expression rules: Step 4A completed (all functions verified)
  - [ ] Interfaces: BOTH Step 4A (functions verified) AND Step 4B (components verified, instruction files loaded)
- [ ] Loaded primary domain reference for this task (Step 2)?
- [ ] Loaded UI pattern examples if applicable (Step 3)?
- [ ] Understand naming conventions (table names, field names, relationship names)?
- [ ] Understand dependency order (what must exist before creating this object)?
- [ ] Have actual UUIDs from environment (not fabricated)?
- [ ] Know which relationships are mandatory (e.g., USER fields → SYSTEM_RECORD_TYPE_USER)?
- [ ] Completed Step 6 pre-implementation verification?

---

##### Step 7A: Generate SAIL Expression

**For Expression Rules:**
- Generate the expression body as a string
- Ensure all `ri!` references match input parameter names
- Apply null safety patterns from null-safety-patterns.md
- Use only verified functions (Step 4A complete)

**For Interfaces:**
- Generate the full interface expression as a string
- Ensure all `ri!` references match input parameter names
- Apply null safety patterns from null-safety-patterns.md
- Use only verified functions AND components (Step 4A + 4B complete)
- Follow interface-generation-checklist.md validation items

---

##### Step 7B: Validate Expression (Retry Loop)

**🛑 MANDATORY: Load validation checkpoint document**

```
Load: references/validation-checkpoint.md
```

**Validation workflow:**

1. **Call `validateExpression` MCP tool** with generated SAIL
2. **If validation passes** (`hasErrors: false`) → Proceed to Step 7C
3. **If validation fails** (`hasErrors: true`) → Enter retry loop:

**Retry Loop (MAX_ATTEMPTS = 3):**

```
Attempt 1: Validate generated SAIL
  ❌ If fails → Extract error details, fix SAIL

Attempt 2: Re-validate fixed SAIL
  ❌ If fails → Extract error details, fix again

Attempt 3: Final validation
  ❌ If fails → Report to user with full error details
  ✅ If passes → Proceed to Step 7C
```

**Error Fix Strategy:**

- **Parse errors** (syntax) → Check delimiters, function names (anti-hallucination list)
- **Discovery errors** (references) → Verify `ri!`, `recordType!`, `local!` names
- **Eval errors** (types) → Apply explicit casting, fix null handling

**After 3 failed attempts:**
- Report validation errors to user
- Include error messages from `parseErrors`, `discoveryErrors`, `evalErrors`
- Request user guidance or manual review

**Validation checkpoint document provides:**
- ✅ Complete retry loop pattern (with pseudo-code)
- ✅ Error fix strategies by error type
- ✅ Real examples (expression rule + interface with retry)
- ✅ Common validation errors and fixes table

---

##### Step 7C: Create Object (Only After Validation Passes)

✅ **Validation passed** → Now safe to call MCP tools:

**For Expression Rules:**
```
Call: createExpressionRule
Parameters: {
  name: "...",
  expression: "<validated SAIL>",
  inputs: [...],
  ...
}
```

**For Interfaces:**
```
Call: createInterface
Parameters: {
  name: "...",
  expression: "<validated SAIL>",
  inputs: [...],
  ...
}
```

**IMPORTANT:** Do NOT call `createExpressionRule` or `createInterface` until Step 7B validation passes.

---

##### Step 7D: Test Conditionally-Rendered Interfaces (`testInterface`)

**Applies to interfaces only.** `createInterface` parses and design-checks the whole expression, but only **evaluates** the branches active under the default inputs (null unless you supply `testInputs`). A runtime error inside a branch that is inactive at creation — e.g., a component missing a required parameter in the `isUpdate = true` path — passes creation cleanly and fails only when a user triggers that branch.

**After creating an interface with any input-dependent rendering, call `testInterface`** with inputs chosen to render each branch the default inputs do NOT exercise, and check `diagnostics.error`:

- Create/update form gated by `isUpdate` → test with `isUpdate: true` (and a sample `record`)
- `showWhen` / `if()` sections gated by a status, role, or flag → test with values that make each section render
- Components fed by a record input (`ri!record`) → test with a populated record

Repeat until every conditionally-rendered branch has been evaluated at least once. **Skip only** when the entire component tree already renders under the default inputs.

See `references/validation-checkpoint.md` ("What createInterface / updateInterface Validate") for the full explanation.

---

## Documentation Lookup Tiers (Overview)

When you need information beyond loaded skill references, use this three-tier approach:

- **Tier 1: Skill References** — Always check first (curated patterns, anti-patterns, examples, 146 components with registry)
- **Tier 2A: functions.json + curl** — For function existence checks and signatures (definitive, fast, 495 functions)
- **Tier 2B: Components registry + functions.json** — For component existence, instruction files, and signatures (146 components)
- **Tier 3: Documentation Search Tool** — For UI/UX patterns, recipes, best practices (semantic search, optional MCP server)

**Quick decision:**
- Need to verify a function exists? → Tier 2A
- Need to verify a component exists? → Tier 1 (registry) → Tier 2B
- Need UI pattern or recipe? → Tier 3
- Need both? → Use Tier 2A for functions, Tier 2B for components, Tier 3 for patterns

**For complete workflows, error handling, curl commands, and examples: Load `references/documentation-lookup-strategy.md`**

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
- Prevention: Load the file corresponding to the object type mentioned in the Resource Reference Map section

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
VERSION="26.6"  # Use the configured version

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
# Returns: "/suite/help/26.6/fnc_system_queryrecordtype.html" if exists
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
1. Default to version 26.6 (latest)
2. Suggest to user: "Using Appian 26.6 docs. To use a different version, update the Configuration section in skills/appian/SKILL.md"
