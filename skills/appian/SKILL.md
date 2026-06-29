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

**Supported Versions (as of 2026-06-25):**
26.6, 26.5, 26.3, 25.4, 25.3, 25.2, 25.1, 24.4, 24.3

Update this version to match your Appian environment. This affects:
- Documentation URL lookups
- Function availability checks
- Version-specific guidance

**To change:** Edit the version number above.

**Redirect behavior:** Appian's documentation server may redirect older version URLs to the current latest version. Using `curl -L` handles this transparently. The "Supported Versions" list represents versions this skill has been validated against; the functions.json URL will follow redirects to whatever Appian considers current.

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
| Executing Tier 2 function lookups (curl commands, error handling) or Tier 3 pattern searches (search strategy, failure modes) | `references/documentation-lookup-strategy.md` |
| User asks "how do I..." / "show me recipes" / needs patterns not in references | Official docs search tool (see Tier 3) |
| Writing any expression or expression rule (ALWAYS load core pattern files) | `references/function-reference.md`, `references/null-safety-patterns.md`, `references/short-circuit-patterns.md` |
| Using Appian functions, operators, or type conversions in expressions | `references/function-reference.md`, `references/null-safety-patterns.md` |
| Need detailed array, date/time, match, or forEach patterns | `references/function-patterns-index.md` (loads pattern files on demand) |
| Querying record types with filters, sorting, relationships, aggregations (KPIs), or paging | `references/query-record-type-patterns.md` |
| Creating or managing an application | `references/applications.md` |
| Creating/modifying record types, fields, relationships, views, or actions | `references/record-types.md` |
| Requirements mention filtering, searching, faceted navigation, or record list dropdowns | `references/record-type-user-filters.md` |
| Creating/modifying interfaces or writing SAIL form expressions | `references/interfaces.md`, `references/expressions.md` |
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
| Auditing interfaces for accessibility, fixing accessibility defects, or building accessible interfaces (WCAG compliance) | `references/accessibility-audit.md`, `references/component-checks.md`, `references/accessibility-reference.md` |

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
- Building interfaces → Load `references/interfaces.md`, `references/expressions.md`, AND `references/sail.md`
- Creating process models → Load `references/process-models.md` AND `references/node-types.md`

**Why expressions.md for interfaces:**
Interfaces contain complex SAIL expressions requiring the same patterns as expression rules:
- UUID-qualified format for `recordType!` references: `'recordType!{uuid}Name.fields.{fieldUuid}fieldName'`
- Expression syntax: operators, type system, casting functions, object reference prefixes (`ri!`, `cons!`, `recordType!`)
- Common patterns: security expressions, local variables (`a!localVariables()`), queries, data transformation, validation
- Null handling and type safety patterns critical for form field expressions
- Performance considerations: query optimization, caching in local variables
- Common pitfalls: null propagation, short-circuit evaluation, list indexing

#### Step 3: [MANDATORY CHECKPOINT] Verify Implementation Knowledge

**STOP. Before writing any code, verify you have complete implementation knowledge.**

**A. For Expression Rules - Verify Functions:**

1. **Check anti-hallucination list** in function-reference.md
   - If listed as non-existent (regexmatch, property(), a!dateTimeValue()) → **STOP**, use alternatives
   
2. **Check function-reference.md** for documented functions
   - If fully documented → Proceed to Step 4
   - If NOT documented or only partially documented → Continue to step 3
   
3. **Look up unknown function via Tier 2** (functions.json + curl):
   - **Load `references/documentation-lookup-strategy.md`** for curl commands, error handling, and version redirects
   - Search functions.json for function name or keyword
   - If found → Fetch signature, proceed to Step 4
   - If Tier 2 fails → Document assumption ("Could not verify function existence"), proceed with Tier 1 only
   
4. **Special case - Custom logic:** If implementing distance/encryption/date math/JSON/XML:
   - **ALWAYS search Tier 2 first** for built-in function BEFORE implementing custom logic
   - Common patterns: distance → a!distanceBetween, hash → a!hash, JSON → a!toJson/a!fromJson

**B. For Interfaces - Verify Patterns:**

1. **Check skill references** (interfaces.md, sail.md, ui-patterns.md)
   - If pattern fully documented → Proceed to Step 4
   - If pattern NOT documented or incomplete → Continue to step 2
   
2. **Search for pattern via Tier 3** (documentation search tool, if available):
   - **Load `references/documentation-lookup-strategy.md`** for search strategy, query examples, and failure modes
   - Search official documentation: "drilldown pattern", "grid filtering", "interface recipes"
   - If found → Combine Tier 3 pattern + Tier 1 anti-patterns
   - If Tier 3 unavailable → Document assumption, proceed with Tier 1 best-effort
   
**After verification:**
- Review Tier 2/3 results alongside Tier 1 skill references
- Combine both sources: skill references provide context/anti-patterns, Tier 2/3 provides official signatures/patterns
- Apply the enriched knowledge in Step 6

**For Q&A (not implementation):** If user asks "Does X exist?" or "What are the parameters?" without requesting implementation, answer using the same Tier 1 → Tier 2/3 workflow but stop after providing the answer.

**Examples (condensed):**

**Implementation Example 1: Expression rule with unknown function**
- Request: "Create an expression rule that calculates distance in miles between two GPS coordinates"
  - **Step 3 [CHECKPOINT]:** Check function-reference.md (not found) → Custom logic detected (distance calculation) → **Automatically proceed to Tier 2** → Load documentation-lookup-strategy.md → Search functions.json for "distance" → Found `a!distanceBetween()`
  - **Result:** 5-line expression vs 40-line custom Haversine formula

**Implementation Example 2: Interface with unknown pattern**
- Request: "Create interface with grid that drills down to order details on row click"
  - **Step 3 [CHECKPOINT]:** Check skill refs (incomplete) → **Automatically proceed to Tier 3** → Load documentation-lookup-strategy.md → Search "drilldown pattern" → Found official recipe
  - **Result:** 1 validation error vs 4 errors when inventing (75% reduction)

**For detailed tier lookup workflows, curl commands, error handling, and more examples, see `references/documentation-lookup-strategy.md`**

#### Step 4: Load Supplementary References
Based on what you discover in Steps 2-3, load additional references:
- Field type constraints → `references/field-types.md`
- Security configuration → `references/security.md`
- Multi-object tasks → `references/change-planning.md`

#### Step 5: Final Pre-Implementation Verification

**Complete these actions before writing code:**

**For Expression Rules:**
1. **Verify functions** (already completed in Step 3)
   - Confirmed: anti-hallucination list checked, unknown functions looked up
2. **Apply null safety patterns**
   - Use patterns from null-safety-patterns.md
   - Guard all operations: `if(a!isNullOrEmpty(value), default, operation)`
3. **Verify type handling**
   - Know input types (from rule parameters) and output type (return value)
   - Use explicit casting where needed (tointeger(), todecimal(), totext())

**For Interfaces:**
1. **Verify patterns** (already completed in Step 3)
   - Confirmed: patterns from Tier 1 or Tier 3, not invented
2. **Verify components**
   - Using only documented SAIL components (a!gridField, a!textField, a!sectionLayout, etc.)
   - Check component-reference.md or components/ directory for parameters
3. **Plan expression extraction**
   - Identify complex logic (calculations, validations, transformations)
   - Plan to extract to expression rules (not inline in interface)

**For Both:**
- **References loaded:** All 5 universal patterns + primary domain reference
- **Domain knowledge ready:** Naming conventions, dependency order, mandatory relationships
- **UUIDs available:** From environment (via list/get operations), not fabricated

#### Step 6: Write Code

After completing Step 5 verification, you have complete implementation knowledge. Write expression rule body or interface SAIL code.

---

## Documentation Lookup Tiers (Overview)

When you need information beyond loaded skill references, use this three-tier approach:

- **Tier 1: Skill References** — Always check first (curated patterns, anti-patterns, examples)
- **Tier 2: functions.json + curl** — For function existence checks and signatures (definitive, fast)
- **Tier 3: Documentation Search Tool** — For UI/UX patterns, recipes, best practices (semantic search)

**Quick decision:**
- Need to verify a function exists? → Tier 2
- Need UI pattern or recipe? → Tier 3
- Need both? → Use Tier 2 for functions, Tier 3 for patterns

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

