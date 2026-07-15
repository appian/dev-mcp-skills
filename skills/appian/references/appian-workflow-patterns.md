# Appian Interface Workflow Patterns

**Purpose:** Understand how Appian interface types connect in user workflows and when to use each type.

**When to load:** When creating any interface (Dashboard, Summary View, Form, Record Action Form) to understand workflow context and type selection.

---

## Standard User Workflow

**Standard user workflow in Appian applications:**

```
Dashboard (high-level view)
  ├─ Create action → Form (create mode, isUpdate=false)
  └─ Click record → Summary View (read-only, shows ONE record + related records)
      └─ Record actions → Form (update mode, isUpdate=true)
```

This workflow shows how interfaces connect:
1. User starts at **Dashboard** (lists multiple records)
2. Clicks record → navigates to **Summary View** (read-only details)
3. Clicks action ("Edit") → launches **Form** (editable)

---

## Interface Type Definitions

### Dashboard Interface

**Purpose:** High-level view of business/subject matter  
**Layout:** `a!headerContentLayout()`  
**Contains:** KPIs, reports (as sections), grids with records  
**Actions:** Create new records (not tied to specific record)  
**NOT for:** Record-specific details (use Summary View instead)

**Example (Case Management):**
- "Case Manager Dashboard displays open cases grid, overdue cases count, cases by status chart"
- Has "Create Case" record action → launches Case Form (create mode)

---

### Summary View Interface (Record View)

**Purpose:** Display ONE specific record + related records (1-to-many)  
**Layout:** `a!headerContentLayout()`  
**Initially:** **Read-only display** (use display components, NOT input fields)  
**Contains:** Record details, related records (e.g., Notes for this Case, Audit Trail for this Order)  
**Actions:** Record-specific actions (Update, Delete, Approve, Reject, Certify)  
**When action triggered:** Form opens or sections become editable

**Example (Case Management):**
- "Case Summary View shows case details, related notes, audit trail"
- Uses `a!richTextDisplayField()`, `a!tag()`, `a!stampField()` for read-only display
- Has "Update" record action → launches Case Form (update mode)
- Has "Close Case" record action → launches Close Case Form

**Critical Rule:** Summary Views are **read-only initially**. Do NOT use input fields (`a!textField()`, `a!dropdownField()`) for display.

---

### Form Interface

**Purpose:** Create OR update records (same form handles both modes)  
**Layout:** `a!formLayout()`  
**Triggered by:** Create action from Dashboard OR record action from Summary View  
**Linked to:** Process Model (with start_form_interface)  
**Important:** Use ONE form for both create and update (distinguish with isUpdate flag)

**Example (Case Management):**
- "Case Form" handles both:
  - Create Case (from Dashboard "Create" action, `isUpdate=false`)
  - Update Case (from Summary View "Update" action, `isUpdate=true`)
- Uses `a!textField()`, `a!dropdownField()`, etc. for editable fields
- Has Submit + Cancel buttons

**Critical Rule:** Forms are **editable**. Use input fields, NOT display components.

---

### Form Consolidation Rule

**Use ONE form for both create and update:**

```sail
a!formLayout(
  label: if(ri!isUpdate, "Edit Case", "New Case"),
  contents: {
    /* Fields here - some may be read-only in update mode */
    a!textField(
      label: "Case ID",
      value: ri!record['recordType!Case.fields.id'],
      readOnly: ri!isUpdate  /* Lock primary key in update mode */
    ),
    /* Other fields */
  },
  buttons: a!buttonLayout(
    primaryButtons: a!buttonWidget(
      label: if(ri!isUpdate, "Save Changes", "Create Case"),
      submit: true
    ),
    secondaryButtons: a!buttonWidget(
      label: "Cancel",
      value: true,
      saveInto: ri!cancel,
      submit: true,
      validate: false
    )
  )
)
```

**Benefits:**
- ✅ Single source of truth for field definitions
- ✅ Consistent validation rules
- ✅ Easier maintenance
- ✅ Process model controls mode via `isUpdate` parameter

---

### Report/Section Consolidation Rules

**Reports are sections WITHIN interfaces, NOT standalone:**
- If report displays persona-relevant data → embed as section in Dashboard
- If report shows record-specific data → embed as section in Summary View
- Only create standalone report if requirements explicitly name it as separate interface

**Example - Consolidate:**
- ❌ WRONG: "Order Revenue Report" as standalone interface
- ✅ CORRECT: Revenue Report section within "Order Dashboard"

**Example - Standalone OK:**
- Requirements: "Monthly Sales Report interface allows managers to view aggregated sales data"
- ✅ CORRECT: Standalone "Monthly Sales Report" (explicitly named interface)

---

## Surfacing Record Actions on Interfaces

**Configuring a record action on a record type is only half the job — the action will not appear to users until it is placed where they can reach it.** After creating a `LIST_ACTION` or `RELATED_ACTION` (see `references/record-types.md`), you must surface it on the appropriate interface(s). How you do this depends on the action type and where records are being displayed.

### List actions

`LIST_ACTION`s are **not tied to a specific record** — they are most often used for **Create** operations ("New Case", "New Order").

- List actions automatically appear above Appian's **built-in record list**. However, the built-in record list is now **rarely used** — most modern applications display lists of records with a custom `a!gridField()` on a Dashboard or landing page instead.
- **On a custom `a!gridField()`, list actions do NOT appear automatically.** They must be surfaced explicitly with the grid's `recordActions` parameter, or users will never see them.

```
a!gridField(
  label: "Cases",
  data: a!recordData(recordType: 'recordType!{rtUuid}Case'),
  columns: { ... },
  recordActions: {
    a!recordActionItem(action: 'recordType!{rtUuid}Case.actions.{createCaseKey}')
  }
)
```

### Related actions

`RELATED_ACTION`s operate on a **specific record** (Edit, Delete, Approve, status transitions). On a **record view** (Summary View), there are **three** ways a related action can appear:

1. **Automatic related-actions view (tab).** If the record type's `hideRelatedActionsView` is set to `false`, Appian generates a separate tab listing all related actions. **Modern applications almost always set `hideRelatedActionsView: true`** — the separate tab is a clumsy UX and is generally avoided.

2. **Related action shortcut on the view** (`relatedActionShortcuts`). This is the **most common** way to configure related actions on a record view — the action is surfaced as a shortcut on the view itself, no SAIL required.

3. **Explicit `a!recordActionField()` in the interface.** On record views this is used mainly on **complex interfaces**, to position a related action **immediately next to the page content it relates to** (rather than in a shortcut bar). Reserve it for that purpose on views — prefer shortcuts (#2) for standard placement.

```
a!recordActionField(
  actions: {
    a!recordActionItem(
      action: 'recordType!{rtUuid}Case.actions.{editCaseKey}',
      /* identifier is REQUIRED for related actions — the record's primary key.
         On a record view use rv!record[...id]; in a grid row use fv!row[...id]. */
      identifier: rv!record['recordType!{rtUuid}Case.fields.{idUuid}id']
    )
  }
)
```

### Related actions outside a record view

When records are shown **outside a record view** — most notably **grid rows** on a Dashboard or landing page — related actions **do NOT appear automatically**. You must surface them explicitly, either via:

- the grid's `recordActions` parameter (renders as an action on the grid), or
- `a!recordActionField()` placed in a column or elsewhere in the interface.

**Rule of thumb:** if you configure a `RELATED_ACTION` and expect users to reach it from a custom grid, you must wire it up — it will not surface on its own.

> See `references/component-reference.md` for `a!recordActionField()` / `a!recordActionItem()` signatures, and `references/sail.md` for grid `recordActions` examples.

---

## Common Anti-Patterns

### ANTI-PATTERN #1: Using Form for "Details" Pages

**DO NOT use Form interface type for viewing record details.**

❌ **WRONG:**
- User wants to view case details
- Create "Case Details Form" with `a!formLayout()` and input fields
- Result: All fields look editable when they should be read-only

✅ **CORRECT:**
- Create "Case Summary View" (Record View interface type)
- Use `a!headerContentLayout()` with display components
- Add "Edit Case" record action that launches "Case Form"

**Test:** If the primary purpose is to **view** a single record, use Summary View (not Form).

---

### ANTI-PATTERN #2: Multiple Forms for Create + Update

**DO NOT create separate forms for create and update.**

❌ **WRONG:**
- Create "Create Case Form" for new cases
- Create "Edit Case Form" for updating cases
- Result: Duplicate logic, inconsistent validation, maintenance burden

✅ **CORRECT:**
- Create ONE "Case Form" with `isUpdate` input (Boolean)
- Use `if(ri!isUpdate, "Edit Case", "New Case")` for dynamic title
- Process Model sets `isUpdate: false` for create, `isUpdate: true` for update
- Some fields may be read-only in update mode (e.g., primary key, created date)

**Test:** If two interfaces do the same thing with slight variations, consolidate into ONE with parameters.

---

### ANTI-PATTERN #3: Editable Fields in Summary Views

**DO NOT use input fields in Summary Views.**

❌ **WRONG:**
- Summary View uses `a!textField()`, `a!dropdownField()` for display
- Fields have `readOnly: true` parameter
- Result: Fields look editable but don't save, confusing UX, unnecessary validation

✅ **CORRECT:**
- Summary View uses `a!richTextDisplayField()`, `a!tag()`, `a!stampField()`, `a!richTextItem()`
- Result: Clear read-only display, no validation overhead
- Actions trigger forms for edits

**Component Selection:**
- **Summary View (read-only):** `a!richTextDisplayField()`, `a!tag()`, `a!stampField()`, `a!cardLayout()`
- **Form (editable):** `a!textField()`, `a!dropdownField()`, `a!integerField()`, `a!dateField()`

---

### ANTI-PATTERN #4: Functional Requirement ≠ Dedicated Interface

**DO NOT create interfaces for functional requirements alone.**

Functional requirements describe WHAT the system does. Interface requirements describe HOW users interact via specific UI.

**"System displays/provides" ≠ Dedicated Interface:**

❌ **WRONG quotes** (functionality only):
- "System displays revenue calculations" → Do NOT create "Revenue Report"
- "System provides comparison of data" → Do NOT create "Comparison Report"
- "System calculates totals" → Do NOT create "Calculator Interface"

**Why wrong:** Describes what system does, not a specific interface. Embed in existing Dashboard/Summary View.

✅ **CORRECT quotes** (names interface):
- "**Revenue Report** displays system calculations" → Create Report
- "**Comparison Dashboard** shows month-over-month changes" → Create Dashboard
- "User accesses **Historical Data Viewer** to compare periods" → Create Viewer

**Test Before Creating Interface:**
1. Does requirement name the interface? ("Order Report", "Case Dashboard")
2. Does it use interface keywords? (dashboard, form, report, interface, screen, page)
3. Can functionality be embedded in existing interface as section?

**If NO to 1-2 and YES to 3:** Embed as section in Dashboard/Summary View

**Examples:**

❌ WRONG: "System tracks order fulfillment" → Create "Order Tracker Dashboard"  
**Why wrong:** Describes capability. Embed as section in Order Dashboard.

❌ WRONG: "System displays audit trail" → Create "Audit Trail Report"  
**Why wrong:** Audit trail is record-specific. Embed as section in Summary View.

✅ CORRECT: "**Order Tracker Dashboard** displays fulfillment status by customer" → Create Dashboard  
**Why correct:** Explicitly names dashboard interface

---

### ANTI-PATTERN #5: Inline Features ≠ Separate Interface

**DO NOT create separate interfaces for instructions, help, or validations.**

These are inline features (field-level help, tooltips, validation rules), not separate interfaces.

**Examples:**

❌ WRONG: "Provide in-app instructions" → Create "Instructions Interface"  
❌ WRONG: "Validate email format" → Create "Validation Interface"

✅ CORRECT: Document inline help/validation requirements in form descriptions

---

## Appian Platform Knowledge

### Platform-Provided Features

The following features are provided by Appian's platform and should NOT be custom-built:

- **User Context Display** — Username, full name, profile photo are shown in Appian's standard header
- **Language Selection** — Multi-language switching is available via user preferences
- **Site-Level Navigation** — Application header, site navigation menus, breadcrumbs are platform-managed
- **Authentication/Login** — Appian has built-in authentication (SSO, LDAP)
- **User Management** — Appian has native user/group administration (unless requirements explicitly request custom screens)
- **Role/Permission Management** — Appian handles security groups natively (unless requirements explicitly request custom screens)
- **Document Management** — Use Appian's native document storage (don't design custom file storage interfaces)

**Note:** Within-interface navigation components (wizard steps, tabs, accordions using SAIL components like `a!wizardLayout()`, `a!sectionLayout()`, `a!columnsLayout()`) are legitimate interface elements and should be included when requirements specify them.

---

## Interface Type Selection Checklist

Before creating an interface, answer these questions:

**Question 1: What is the user doing?**

- [ ] Viewing details of ONE existing record? → **Summary View** (read-only, `a!headerContentLayout()`, display components)
- [ ] Creating a NEW record OR editing an existing record? → **Form** (editable, `a!formLayout()`, input fields)
- [ ] Viewing high-level overview across multiple records? → **Dashboard** (KPIs + grids, `a!headerContentLayout()`)
- [ ] Performing specific action on existing record? → **Record Action Form** (context + action fields, `a!formLayout()`)

**Question 2: Does it fit the workflow pattern?**

- [ ] Dashboard → Summary View → Form (standard workflow)
- [ ] Standalone report (explicitly named in requirements)
- [ ] Landing page (entry point for persona)

**Question 3: Can it be consolidated?**

- [ ] Create + update forms → ONE form with `isUpdate` parameter
- [ ] Report → embed as section in Dashboard/Summary View
- [ ] Multiple similar interfaces → ONE with parameters

**If uncertain:** Default to Summary View for viewing, Form for editing.

---

## Related References

- **For interface creation workflow:** `references/interfaces.md`
- **For configuring record actions (LIST_ACTION / RELATED_ACTION):** `references/record-types.md`
- **For Summary View patterns:** `references/record-summary-views.md`
- **For UI patterns (KPIs, grids, sections):** `references/ui-patterns.md`
- **For form save patterns:** `references/write-records-patterns.md`
- **For SAIL syntax:** `references/sail.md`
