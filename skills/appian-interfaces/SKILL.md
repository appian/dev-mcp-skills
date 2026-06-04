---
name: "appian-interfaces"
description: "Build and modify Appian interfaces with good UX patterns. Covers interface object lifecycle, dashboard and summary view design, form organization, interface inputs, and naming conventions. Use when creating dashboards, forms, summary views, or any user-facing interface."
---

## Relevant Tools

Interface tools cover the full lifecycle:

- **Create an interface** — provide a name, optional description, and optionally a SAIL expression
- **List and get interfaces** — retrieve interfaces by UUID or search with optional filtering; scope to an application with `appUuid`
- **Update an interface** — modify name or description on an existing interface
- **Delete an interface** — permanently remove an interface by UUID
- **Get and update SAIL expression** — retrieve or replace the SAIL expression that defines the interface's UI
- **List, add, update, and delete inputs** — manage the interface's input parameters (the `ri!` variables that callers pass in)

## Creating Interfaces

### Naming Conventions

- **Prefix**: Use the application prefix followed by an underscore (e.g., `ACME_`)
- **Pattern**: `[PREFIX]_[InterfacePurpose]` in Title Case with no spaces
- **Examples**:
  - `ACME_Dashboard` — main application dashboard
  - `ACME_CaseSummary` — summary view for a Case record
  - `ACME_CaseForm` — create/update form for a Case record
  - `ACME_CaseActionReassign` — record action form for reassigning a case
  - `ACME_CaseGrid` — reusable grid component for cases

### Interface Types

Interfaces serve different purposes. Choose the right type based on what the user will see:

| Type | Top-Level Layout | Purpose |
|---|---|---|
| Dashboard | `a!headerContentLayout()` | Landing page with KPIs, grids, and navigation |
| Summary View | `a!headerContentLayout()` | Record detail display with related data and actions |
| Form | `a!formLayout()` | Data entry for creating or updating records |
| Record Action Form | `a!formLayout()` | Form used inside a process model for record actions |
| Wizard | `a!wizardLayout()` | Multi-step data entry |
| Reusable Component | No top-level layout | Fragment embedded in other interfaces |

### Operation Sequence

When creating an interface with inputs:

1. Create the interface object (name, description)
2. Add input parameters — each input needs a name and type
3. Set the SAIL expression — the expression can reference the inputs you just added via `ri!` prefix

When updating an existing interface:

1. Get the current interface to see its metadata
2. List current inputs to see what's already defined
3. Get the current SAIL expression
4. Make your changes — add/update/remove inputs, then update the expression

## Interface Input Patterns

### Standard Form Inputs

Forms used as process model start forms or task forms follow a standard input pattern:

| Input Name | Type | Purpose |
|---|---|---|
| `record` | Record Type (or specific entity name like `case`) | The record data being created or edited |
| `isUpdate` | Boolean | Controls create vs. edit mode — `true` for editing existing records, `false` for new records |
| `cancel` | Boolean | Set to `true` by the Cancel button so the process model can skip the record write |

The `record` input carries the data in and out of the form. The process model passes an empty record for creates and a populated record for updates. The form saves field values back into this input, and the process model writes it to the data source after the form submits.

### Dashboard Inputs

Dashboards typically have no inputs or minimal context inputs:

| Input Name | Type | Purpose |
|---|---|---|
| (none) | — | Most dashboards query their own data |
| `appUuid` | Text | Optional — scope dashboard queries to an application |

### Summary View Inputs

Summary views receive the record to display:

| Input Name | Type | Purpose |
|---|---|---|
| `record` | Record Type | The record to display |
| `identifier` | Integer | The record ID (alternative to passing the full record) |

### Record Action Form Inputs

Record action forms are used inside process models triggered from record views. They follow the same pattern as standard forms but are scoped to a specific action:

| Input Name | Type | Purpose |
|---|---|---|
| `record` | Record Type | The record being acted on |
| `cancel` | Boolean | Cancel flow control |

No `isUpdate` input — record action forms always operate on an existing record.

### Input Type QNames

When adding inputs programmatically, use the Appian type QName format:

| Friendly Type | QName |
|---|---|
| Text | `{http://www.appian.com/ae/types/2009}Text` |
| Integer | `{http://www.appian.com/ae/types/2009}Integer` |
| Boolean | `{http://www.appian.com/ae/types/2009}Boolean` |
| Date | `{http://www.appian.com/ae/types/2009}Date` |
| DateTime | `{http://www.appian.com/ae/types/2009}DateTime` |
| User | `{http://www.appian.com/ae/types/2009}User` |

For record type inputs, use the record type's type QName (retrieved from the record type object).

## UX Patterns

### Dashboard Pattern

A dashboard is the application's landing page. It gives users an at-a-glance view of key metrics and quick access to their work.

**Structure**:
1. **Header** — application name, optional subtitle or context
2. **KPI row** — 3-5 metric cards in a columns layout showing counts, totals, or status breakdowns
3. **Primary grid** — the main work queue or record list, filterable and sortable
4. **Quick actions** — links or buttons to start common workflows

**Section organization**:
- Use `a!columnsLayout()` with equal-width columns for the KPI row
- Each KPI card uses `a!cardLayout()` containing a stamp or rich text with the metric value
- Place the primary grid below the KPI row in its own section
- Add navigation links to other views (e.g., "View All Cases", "My Open Tasks")

### Summary View Pattern

A summary view displays a single record's details with related data and available actions.

**Structure**:
1. **Header section** — record title, status badge, key identifiers
2. **Detail sections** — grouped fields showing record data, organized by business domain
3. **Related data** — grids or lists showing child/related records
4. **Action buttons** — available actions for this record (edit, reassign, close, etc.)

**Section organization**:
- Group related fields into sections with descriptive labels (e.g., "Contact Information", "Case Details", "Assignment")
- Use two-column layouts within sections for label-value pairs
- Use `a!sideBySideLayout()` for compact label-value display
- Place related record grids in their own sections below the detail sections
- Put action buttons in a prominent location — either in the header area or as a dedicated actions section

### Form Pattern

Forms collect data for creating or updating records. A single form handles both create and update modes using the `isUpdate` input.

**Structure**:
1. **Form title** — dynamic based on mode: "New [Entity]" or "Edit [Entity]"
2. **Field sections** — organized by business domain, not by data type
3. **Buttons** — Submit (primary) and Cancel (secondary)

**Section organization**:
- Group fields by business meaning, not by field type (don't put all text fields together)
- Put the most important fields first — users scan top to bottom
- Use two-column layouts for forms with many fields to reduce scrolling
- Place optional fields after required fields within each section
- Use collapsible sections for advanced or rarely-used fields
- Keep forms focused — if a form has more than 3-4 sections, consider whether it should be a wizard instead

**Field organization within sections**:
- Required fields before optional fields
- Related fields adjacent to each other (e.g., first name next to last name)
- Dropdown/selection fields that affect other fields' visibility should come before the fields they control
- Date ranges together (start date, end date)
- Address fields in standard order (street, city, state, zip)

### Record Action Form Pattern

Record action forms are specialized forms used inside process models triggered from record views. They handle a specific action on an existing record (reassign, close, escalate, etc.).

**Structure**:
1. **Context display** — show key record fields as read-only so the user knows what they're acting on
2. **Action fields** — the fields specific to this action (e.g., new assignee, resolution notes, escalation reason)
3. **Buttons** — action-specific primary button label (e.g., "Reassign", "Close Case") and Cancel

**Key differences from standard forms**:
- No `isUpdate` input — always operating on an existing record
- Fewer fields — only what's needed for the action, not the full record
- Read-only context — show enough record detail for the user to confirm they're acting on the right record
- Action-specific button labels — "Reassign" not "Submit", "Close Case" not "Save"

## Conventions

### One Interface Per Purpose

Create separate interfaces for each distinct user interaction:
- One dashboard interface per application
- One summary view per record type that needs a detail view
- One form per record type (handles both create and update)
- One interface per record action (reassign, close, escalate, etc.)

Don't combine unrelated views into a single interface with mode switching beyond create/update.

### Interface Descriptions

Write descriptions that explain the interface's purpose and where it's used:
- "Dashboard for the Case Management application. Displays KPI metrics and the active cases grid."
- "Create and update form for Case records. Used as a start form in the Create Case process model."
- "Summary view for Case records. Displays case details, related notes, and available actions."

### SAIL Expression Management

When setting or updating a SAIL expression on an interface:
- The expression must start with `=`
- Reference inputs using the `ri!` prefix matching the input names you defined
- For SAIL syntax guidance (components, layouts, data binding, interaction patterns), activate the SAIL knowledge skill

## Common Pitfalls

- **Using plain-text record type names in SAIL expressions** — all `recordType!` references in expressions submitted via `updateInterfaceExpression` must use the UUID-qualified format wrapped in single quotes: `'recordType!{rtUuid}Name.fields.{fieldUuid}fieldName'`. Plain-text names like `recordType!Board Committee Submission.fields.status` fail at parse time. See the Appian Expressions skill for the full format specification.
- **Setting the SAIL expression before adding inputs** — the expression will fail validation if it references `ri!` variables that don't exist yet; add inputs first, then set the expression
- **Mismatched input names** — the `ri!` references in the SAIL expression must exactly match the input names defined on the interface object; a typo causes a runtime error
- **Using the wrong top-level layout** — `a!formLayout()` for forms (needs buttons for submit), `a!headerContentLayout()` for dashboards and summary views; mixing them up breaks the UX
- **Overloading a single interface** — cramming dashboard, summary, and form into one interface with complex mode switching; use separate interfaces for separate purposes
- **Forgetting the Cancel button on forms** — forms used in process models need a Cancel button that saves `true` into `ri!cancel` so the process can handle cancellation
- **Not passing `isUpdate` correctly** — the process model must pass `true` for edit and `false` for create; if the form always shows "New [Entity]", check that the process model is setting this input
- **Putting too many fields on one form** — if a form has more than 15-20 fields, consider splitting into sections with collapsible areas or using a wizard layout
- **Forgetting to scope dashboard queries** — dashboard grids should filter to relevant records (e.g., current user's cases, open items) rather than showing all records unfiltered

## UX Pattern Reference

Load `references/ui-patterns.md` for detailed UX patterns including dashboard layouts, summary view structures, form organization templates, and section grouping conventions.

## When You Need More

For SAIL syntax — components, layouts, data binding, interaction patterns — activate the SAIL knowledge skill. This skill focuses on interface design and UX; SAIL covers how to write the code.

For questions about specific Appian interface behavior or advanced patterns beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.
