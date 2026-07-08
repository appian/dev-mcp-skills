# Interfaces

## Create JSON Schema

```json
{
  "name": "CM_CaseForm",
  "description": "Create and update form for Case records",
  "inputs": [
    {"name": "record", "type": "record-type-reference-here"},
    {"name": "isUpdate", "type": "Boolean"},
    {"name": "cancel", "type": "Boolean"}
  ],
  "expression": "=a!formLayout(titleBar: if(ri!isUpdate, \"Edit Case\", \"New Case\"), ...)"
}
```

## Naming Conventions

- **Prefix**: Application prefix + underscore (e.g., `CM_`)
- **Pattern**: `PREFIX_InterfacePurpose` in Title Case, no spaces
- **Examples**:
  - `CM_Dashboard` — main application dashboard
  - `CM_CaseSummary` — summary view for a Case record
  - `CM_CaseForm` — create/update form for Cases
  - `CM_CaseActionReassign` — record action form
  - `CM_CaseGrid` — reusable grid component

## Interface Types

| Type | Top-Level Layout | Purpose | Editability |
|---|---|---|---|
| Dashboard | `a!headerContentLayout()` | Landing page with KPIs, grids, navigation | Read-only (display components) |
| Summary View | `a!headerContentLayout()` | Record detail display with actions | **Read-only** (display components) |
| Form | `a!formLayout()` | Data entry for creating or updating records | **Editable** (input fields) |
| Record Action Form | `a!formLayout()` | Form for record actions (reassign, close) | Editable (input fields) |
| Wizard | `a!wizardLayout()` | Multi-step data entry | Editable (input fields) |
| Reusable Component | No top-level layout | Fragment embedded in other interfaces | Varies by purpose |

---

## Interface Type Decision Tree

**Use this decision tree to choose the correct interface type:**

### Question 1: What is the user doing?

**A. Viewing details of ONE existing record?**
→ **Summary View** (Record View)
- Layout: `a!headerContentLayout()`
- **Read-only initially** (use display components, NOT input fields)
- Components: `a!richTextDisplayField()`, `a!tag()`, `a!stampField()`, `a!cardLayout()`
- Has record actions that launch Forms or Process Models
- Example: "Case Summary View", "Order Details View", "Customer Profile View"

**B. Creating a NEW record OR editing an existing record?**
→ **Form**
- Layout: `a!formLayout()`
- **Editable fields** (use input components)
- Components: `a!textField()`, `a!dropdownField()`, `a!integerField()`, `a!dateField()`
- Has Submit + Cancel buttons
- Use `isUpdate` input to handle both create and update in ONE form
- Example: "Case Form" (handles both create and update), "Order Form"

**C. Viewing high-level overview across multiple records?**
→ **Dashboard**
- Layout: `a!headerContentLayout()`
- KPIs, charts, grids showing multiple records
- Has "Create" actions (not tied to specific record)
- Example: "Case Manager Dashboard", "Sales Dashboard"

**D. Performing specific action on existing record?**
→ **Record Action Form**
- Layout: `a!formLayout()`
- Shows context (read-only), action fields (editable), comments
- Action-specific button label ("Reassign", "Close Case", "Approve")
- Example: "Reassign Case Form", "Close Case Form", "Approve Request Form"

**E. Multi-step data entry process?**
→ **Wizard**
- Layout: `a!wizardLayout()`
- Step-by-step flow with validation between steps
- Example: "Onboarding Wizard", "Order Entry Wizard"

---

### Question 2: Does it fit the standard workflow?

**Standard Appian workflow:**

```
Dashboard (list view)
  ├─ Create action → Form (create mode, isUpdate=false)
  └─ Click record → Summary View (read-only)
      └─ Edit action → Form (update mode, isUpdate=true)
```

**Validation checks:**
- [ ] If interface shows ONE record details → Use Summary View (not Form)
- [ ] If interface creates/edits records → Use Form
- [ ] If interface is triggered FROM Summary View → Likely a Form or Record Action Form
- [ ] If interface is entry point for persona → Likely a Dashboard

---

### Critical Rules

**✅ DO:**
- Use **Summary View** for viewing ONE record (read-only initially)
- Use **Form** for creating or updating records (editable)
- Use ONE Form for both create and update (with `isUpdate` flag)
- Use display components in Summary Views (`a!richTextDisplayField()`, `a!tag()`)
- Use input fields in Forms (`a!textField()`, `a!dropdownField()`)

**❌ DO NOT:**
- Use Form for "details" or "summary" pages → Use Summary View
- Use input fields in Summary Views → Use display components
- Create separate forms for create and update → Use ONE form with `isUpdate`
- Make Summary Views editable without action triggers → Actions launch Forms

---

## Common Anti-Patterns

### ANTI-PATTERN #1: Using Form for "Details" Pages

**The Problem:**

❌ **WRONG:**
- User wants to view case details
- Create "Case Details Form" with `a!formLayout()`
- Use `a!textField()`, `a!dropdownField()` with `readOnly: true`
- Result: Fields look editable but don't save, confusing UX, unnecessary validation overhead

✅ **CORRECT:**
- Create "Case Summary View" (interface type: Record View)
- Use `a!headerContentLayout()` with display components
- Use `a!richTextDisplayField()`, `a!tag()`, `a!stampField()`
- Add "Edit Case" record action that launches "Case Form"
- Result: Clear read-only display, actions trigger forms for edits

**How to Identify:**
- Request mentions: "view", "details", "summary", "display record"
- Purpose is to **VIEW** ONE specific record
- No immediate editing needed

**Correct Choice:** Summary View (not Form)

---

### ANTI-PATTERN #2: Multiple Forms for Create + Update

**The Problem:**

❌ **WRONG:**
- Create "Create Case Form" for new cases (isUpdate=false)
- Create "Edit Case Form" for updating cases (isUpdate=true)
- Result: Duplicate logic, inconsistent validation, maintenance burden

✅ **CORRECT:**
- Create ONE "Case Form" with `isUpdate` input (Boolean)
- Use `if(ri!isUpdate, "Edit Case", "New Case")` for dynamic title
- Process Model sets `isUpdate: false` for create action, `isUpdate: true` for update action
- Some fields may be read-only in update mode (e.g., primary key, created date)

**Example:**

```sail
a!formLayout(
  label: if(ri!isUpdate, "Edit Case", "New Case"),
  contents: {
    a!textField(
      label: "Case ID",
      value: ri!record['recordType!Case.fields.id'],
      readOnly: ri!isUpdate  /* Lock primary key in update mode */
    ),
    a!textField(
      label: "Case Title",
      value: ri!record['recordType!Case.fields.title'],
      required: true
    )
    /* Other fields */
  },
  buttons: a!buttonLayout(
    primaryButtons: a!buttonWidget(
      label: if(ri!isUpdate, "Save Changes", "Create Case"),
      submit: true
    )
  )
)
```

**How to Identify:**
- Two interfaces do the same thing with slight variations
- Same fields, same logic, different mode

**Correct Choice:** ONE Form with `isUpdate` parameter

---

### ANTI-PATTERN #3: Editable Fields in Summary Views

**The Problem:**

❌ **WRONG:**
- Summary View uses `a!textField()`, `a!dropdownField()` for display
- Fields have `readOnly: true` parameter
- Result: Fields look editable, validation still runs, performance overhead, confusing UX

✅ **CORRECT:**
- Summary View uses `a!richTextDisplayField()`, `a!tag()`, `a!stampField()`
- No validation overhead
- Clear read-only appearance
- Actions trigger forms for edits

**Component Selection Guide:**

| Context | Component Type | Examples |
|---------|---------------|----------|
| **Summary View (read-only)** | Display components | `a!richTextDisplayField()`, `a!tag()`, `a!stampField()`, `a!cardLayout()` |
| **Form (editable)** | Input components | `a!textField()`, `a!dropdownField()`, `a!integerField()`, `a!dateField()` |

**How to Identify:**
- Interface shows ONE record
- Purpose is viewing (not editing)
- Has actions like "Edit", "Update", "Close"

**Correct Choice:** Use display components (not input fields with readOnly: true)

## Input Patterns

### Standard Form Inputs

| Input | Type | Purpose |
|---|---|---|
| `record` | Record Type | The record data being created/edited |
| `isUpdate` | Boolean | `true` for edit, `false` for create |
| `cancel` | Boolean | Set by Cancel button for process flow control |

### Summary View Inputs

| Input | Type | Purpose |
|---|---|---|
| `record` | Record Type | The record to display |
| `identifier` | Integer | Record ID (alternative to full record) |

### Record Action Form Inputs

| Input | Type | Purpose |
|---|---|---|
| `record` | Record Type | The record being acted on |
| `cancel` | Boolean | Cancel flow control |

No `isUpdate` — record actions always operate on existing records.

### Dashboard Inputs

Usually none. Dashboards query their own data.

## Input Type References

For built-in types, use the friendly name: `Text`, `Number (Integer)`, `Number (Decimal)`, `Boolean`, `Date`, `Date and Time`, `Time`, `User`, `Group`, `Document`, `Folder`, `Encrypted Text`, `Map`

For record type inputs, use the `typeReference` string from the record type's details (get the record type to find it).

---

## Field Type to Component Mapping

**Purpose:** Choose the correct SAIL component based on the record type field type.

**When to use:** Building forms or summary views that display/edit record type fields.

### Mapping Table

| Field Type | Component | Toggle Parameter | Notes |
|------------|-----------|------------------|-------|
| **TEXT** | `a!textField` | `readOnly` | Use `readOnly: not(local!isEditing)` to toggle. For long text (>255 chars), use `a!paragraphField` |
| **INTEGER** | `a!integerField` | `readOnly` | Use `readOnly: not(local!isEditing)` to toggle. Display: Use `tostring()` for formatting |
| **DECIMAL** | `a!floatingPointField` | `readOnly` | Use `readOnly: not(local!isEditing)` to toggle. Display: Use `fixed()` or `dollar()` for formatting |
| **BOOLEAN** | `a!checkboxField` | `disabled` | Use `disabled: not(local!isEditing)` to toggle. Note: Use `disabled` not `readOnly` (checkboxes don't have readOnly) |
| **DATE** | `a!dateField` | `readOnly` | Use `readOnly: not(local!isEditing)` to toggle. For display-only, use `text(date, "MMM d, yyyy")` in `a!textField` |
| **DATETIME** | `a!dateTimeField` | `readOnly` | Use `readOnly: not(local!isEditing)` to toggle. For display-only, use `text(datetime, "MMM d, yyyy h:mm a")` in `a!textField` |
| **TIME** | `a!textField` | `readOnly` | No TIME input component exists. Use `a!textField` with time value. For display-only, use `text(time, "h:mm a")` |
| **USER** (username) | `a!pickerFieldUsers` | `readOnly` | Use `readOnly: not(local!isEditing)` to toggle. When read-only, shows username. For formatted name display, use `a!textField(readOnly: true)` with `user(username, "firstName") & " " & user(username, "lastName")` |
| **Relationship to lookup table** (Priority, Status) | `a!dropdownField` | `disabled` | Use `disabled: not(local!isEditing)` to toggle. Note: Use `disabled` not `readOnly` (dropdowns don't have readOnly). Query lookup table for choices. See `dropdown-patterns.md` |

**Key Pattern:** Use the input component with toggle parameter (`readOnly` or `disabled`) set to `not(local!isEditing)` to toggle between view and edit modes. Do NOT use `if()` to swap between display and input components.

**Important:** 
- Most components use `readOnly` parameter
- `a!dropdownField` and `a!checkboxField` use `disabled` parameter (they don't have `readOnly`)
- No editable TIME input component exists - use `a!textField` for TIME fields

### Key Rules

**1. USER Fields Always Use User Picker**

❌ **WRONG - Text field:**
```sail
a!textField(
  label: "Assigned To (username)",
  value: local!assignedTo,
  saveInto: local!assignedTo,
  helpTooltip: "Enter username"
)
```

✅ **CORRECT - User picker:**
```sail
a!pickerFieldUsers(
  label: "Assigned To",
  value: local!assignedTo,
  saveInto: local!assignedTo,
  maxSelections: 1,
  readOnly: not(local!isEditing)  /* Toggle edit mode */
)
```

**Why:**
- Validates username exists
- Shows full name (not just username) in picker
- Type-ahead search
- Prevents typos
- Use `readOnly` parameter to toggle, just like other input components

---

**2. USER Fields Show Full Name (Always Read-Only Fields)**

For USER fields that are **always read-only** (like Created By, Modified By), format the display name:

❌ **WRONG - Shows username:**
```sail
a!textField(
  label: "Created By",
  value: local!case['...createdBy'],  /* Shows "john.smith" */
  readOnly: true
)
```

✅ **CORRECT - Shows full name:**
```sail
a!textField(
  label: "Created By",
  value: if(
    a!isNotNullOrEmpty(local!case['...createdBy']),
    trim(
      user(local!case['...createdBy'], "firstName") & " " & 
      user(local!case['...createdBy'], "lastName")
    ),
    "–"
  ),
  readOnly: true
)
```

**For USER fields with edit toggle (like Assigned To):**
```sail
a!pickerFieldUsers(
  label: "Assigned To",
  value: local!assignedTo,
  saveInto: local!assignedTo,
  maxSelections: 1,
  readOnly: not(local!isEditing)  /* Toggle read-only mode */
)
```

**Note:** When `a!pickerFieldUsers` is read-only, it displays the username. If you want to show the full name in read-only mode, use `a!textField(readOnly: true)` with formatted name instead.

---

**3. Lookup Table Relationships Query from Record Type (Edit Mode)**

❌ **WRONG - Hardcoded:**
```sail
a!dropdownField(
  label: "Priority",
  choiceLabels: {"Low", "Medium", "High"},
  choiceValues: {1, 2, 3}
)
```

✅ **CORRECT - Query from record type:**
```sail
/* In local variables */
local!priorities: a!queryRecordType(
  recordType: 'recordType!{uuid}Priority',
  fields: {id, label},
  filters: a!queryFilter(field: isActive, operator: "=", value: true)
).data,

/* In form */
a!dropdownField(
  label: "Priority",
  choiceLabels: index(local!priorities, "label", {}),
  choiceValues: index(local!priorities, "id", {}),
  value: local!priorityId,
  saveInto: local!priorityId,
  disabled: not(local!isEditing),  /* Toggle edit mode - use disabled, not readOnly */
  required: true
)
```

**Note:** `a!dropdownField` uses `disabled` parameter, not `readOnly` (dropdowns don't have readOnly property).

**See `dropdown-patterns.md` for complete pattern.**

---

**4. DATE/DATETIME Fields Format for Display (Read-Only Mode)**

❌ **WRONG - Shows raw ISO format:**
```sail
a!textField(
  label: "Created Date",
  value: local!case['...createdAt'],  /* Shows "2026-07-01T12:34:56Z" */
  readOnly: true
)
```

✅ **CORRECT - Formatted display:**
```sail
a!textField(
  label: "Created Date",
  value: if(
    a!isNotNullOrEmpty(local!case['...createdAt']),
    text(local!case['...createdAt'], "MMM d, yyyy h:mm a"),  /* Shows "Jul 1, 2026 12:34 PM" */
    "N/A"
  ),
  readOnly: true
)
```

---

**5. BOOLEAN Fields Show Yes/No (Read-Only Mode)**

❌ **WRONG - Shows true/false:**
```sail
a!textField(
  label: "Is Active",
  value: local!case['...isActive'],  /* Shows "true" */
  readOnly: true
)
```

✅ **CORRECT - Shows Yes/No:**
```sail
a!textField(
  label: "Is Active",
  value: if(local!case['...isActive'], "Yes", "No"),
  readOnly: true
)
```

---

### Pattern: Form with Edit/Cancel Toggle (Using readOnly)

**Use the `readOnly` parameter to toggle edit mode, NOT separate components with `if()`.**

```sail
a!localVariables(
  /* Query the record */
  local!case: a!queryRecordByIdentifier(...),
  
  /* Query lookup tables */
  local!priorities: a!queryRecordType('recordType!Priority', ...).data,
  local!statuses: a!queryRecordType('recordType!Status', ...).data,
  
  /* Track edit mode */
  local!isEditing: false,
  
  /* Editable copies */
  local!title: a!defaultValue(local!case['...title'], ""),
  local!priorityId: a!defaultValue(local!case['...priorityId'], null),
  local!assignedTo: a!defaultValue(local!case['...assignedTo'], null),
  
  a!formLayout(
    contents: {
      /* ID field (always read-only) */
      a!integerField(
        label: "Case ID",
        value: local!case['...id'],
        readOnly: true
      ),
      
      /* TEXT field */
      a!textField(
        label: "Title",
        value: local!title,
        saveInto: local!title,
        readOnly: not(local!isEditing),
        required: true
      ),
      
      /* Lookup table relationship (Priority) */
      a!dropdownField(
        label: "Priority",
        placeholder: "Select a priority",
        choiceLabels: index(local!priorities, "label", {}),
        choiceValues: index(local!priorities, "id", {}),
        value: local!priorityId,
        saveInto: local!priorityId,
        disabled: not(local!isEditing),  /* Use disabled, not readOnly */
        required: true
      ),
      
      /* USER field */
      a!pickerFieldUsers(
        label: "Assigned To",
        value: local!assignedTo,
        saveInto: local!assignedTo,
        maxSelections: 1,
        readOnly: not(local!isEditing)
      ),
      
      /* USER field display (Created By - always read-only) */
      a!textField(
        label: "Created By",
        value: if(
          a!isNotNullOrEmpty(local!case['...createdBy']),
          trim(user(local!case['...createdBy'], "firstName") & " " & user(local!case['...createdBy'], "lastName")),
          "–"
        ),
        readOnly: true
      ),
      
      /* DATETIME field (always read-only) */
      a!textField(
        label: "Created Date",
        value: if(
          a!isNotNullOrEmpty(local!case['...createdAt']),
          text(local!case['...createdAt'], "MMM d, yyyy h:mm a"),
          "N/A"
        ),
        readOnly: true
      )
    },
    buttons: a!buttonLayout(
      primaryButtons: if(
        local!isEditing,
        {
          a!buttonWidget(
            label: "Save",
            saveInto: {
              a!writeRecords(...),
              a!save(local!isEditing, false)
            },
            style: "SOLID"
          )
        },
        {
          a!buttonWidget(
            label: "Edit",
            saveInto: a!save(local!isEditing, true),
            style: "SOLID"
          )
        }
      ),
      secondaryButtons: if(
        local!isEditing,
        {
          a!buttonWidget(
            label: "Cancel",
            saveInto: {
              /* Reset to original values */
              a!save(local!title, a!defaultValue(local!case['...title'], "")),
              a!save(local!priorityId, a!defaultValue(local!case['...priorityId'], null)),
              a!save(local!assignedTo, a!defaultValue(local!case['...assignedTo'], null)),
              a!save(local!isEditing, false)
            },
            style: "OUTLINE"
          )
        },
        {}
      )
    )
  )
)
```

**Key Points:**
- ✅ Use `readOnly: not(local!isEditing)` to toggle edit mode
- ✅ ONE component per field, not two components with `if()`
- ✅ Fields that are always read-only use `readOnly: true`
- ✅ Simpler code, better UX (fields visually appear read-only)
- ✅ Cancel button resets all editable fields to original values

---

## Operation Sequence

**Creating with inputs and expression:**
1. Generate SAIL expression (complete Steps 1-6 from SKILL.md)
2. **Validate expression BEFORE creating** (Step 7B - validation checkpoint):
   - Call `validateExpression` MCP tool with generated SAIL
   - If validation fails → Fix errors and retry (up to 3 attempts)
   - Load `references/validation-checkpoint.md` for complete retry workflow
3. **Only after validation passes** → Call `createInterface` with:
   - `inputs` array (interface parameters)
   - `expression` (validated SAIL expression)
   - Expression references inputs via `ri!` prefix matching input names

**Updating an existing interface:**
1. Get the interface to see current state
2. Modify inputs or expression as needed
3. **Validate modified expression** (same Step 7B workflow):
   - Call `validateExpression` with modified SAIL
   - Fix errors if validation fails (retry up to 3 times)
4. **Only after validation passes** → Call `updateInterface` with modified payload

**Important:** Always validate expressions before creating or updating. This catches syntax/reference/type errors early and enables automatic retry.

## UX Patterns

### Dashboard
1. Header — app name, subtitle
2. KPI row — 3-5 metric cards in `a!columnsLayout()`
3. Primary grid — main work queue, filterable
4. Quick actions — links to common workflows

### Summary View
1. Header — record title, status badge, key IDs
2. Detail sections — grouped fields by business domain
3. Related data — grids of child/related records
4. Action buttons — edit, reassign, close

### Form
1. Title — dynamic: "New [Entity]" or "Edit [Entity]"
2. Field sections — grouped by business meaning
3. Buttons — Submit (primary) + Cancel (secondary)

### Record Action Form
1. Context display — key record fields as read-only
2. Action fields — only what's needed for the action
3. Buttons — action-specific label ("Reassign", "Close Case") + Cancel

## Conventions

- One interface per purpose (don't combine unrelated views)
- Create separate interfaces for: dashboard, each summary view, each form, each record action
- Forms with Cancel need the cancel button to save `true` into `ri!cancel`
- Required fields before optional within sections
- Group fields by business meaning, not by data type

## Common Pitfalls

- **Setting expression before inputs** — expression fails if it references `ri!` variables that don't exist yet; provide both together or add inputs first
- **Mismatched `ri!` names** — expression references must exactly match input names
- **Wrong top-level layout** — `a!formLayout()` for forms, `a!headerContentLayout()` for dashboards/summaries
- **Plain-text record type names in expressions** — use UUID-qualified format: `'recordType!{uuid}Name.fields.{fieldUuid}fieldName'`
- **Forgetting Cancel button on forms** — process model forms need Cancel to save into `ri!cancel`
- **Overloading a single interface** — keep dashboard, summary, and form separate

## Test Cases

Test cases save a set of input values against an interface so the platform can re-render it with those inputs and verify no rendering errors occur. Interface test cases always use NO_ERRORS assertion mode — they pass if the interface renders cleanly, fail if it throws a runtime error.

Create test cases after the interface expression is set, covering each meaningful input combination (e.g. different modes or states the interface can be in).

### Flow

1. Create test cases for the interface after the expression is saved
2. Run all test cases and check for rendering errors
3. If any fail, fix the expression or the test case inputs, then re-run

## SAIL Guidance

For SAIL syntax, components, layouts, and interaction patterns, load `references/sail.md`.
