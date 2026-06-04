# Interface UX Patterns

## Dashboard Pattern

### Purpose
The application dashboard is the user's home base. It answers: "What needs my attention?" and "How is the work going?"

### Layout Structure

```
a!headerContentLayout()
├── header: a!cardLayout() or billboard with app title
└── contents:
    ├── Section: KPI Metrics
    │   └── a!columnsLayout() with 3-5 equal columns
    │       └── Each column: a!cardLayout() with metric
    ├── Section: Primary Work Queue
    │   └── a!gridField() with a!recordData()
    │       └── Filtered to current user or team
    └── Section: Quick Actions (optional)
        └── Links to common workflows
```

### KPI Card Pattern

Each KPI card shows a single metric with visual emphasis:

- Use `a!cardLayout()` as the container
- Use `a!stampField()` with an icon for visual identity (e.g., `"briefcase"` for cases, `"users"` for people, `"clock"` for pending items)
- Display the metric value prominently using `a!richTextDisplayField()` with large, bold text
- Add a label below the value describing what it measures
- Optionally make the card clickable (via `link` parameter) to navigate to a filtered view

**Metric examples**:
- Open cases count
- Cases assigned to current user
- Overdue items
- Average resolution time
- Items awaiting approval

### Primary Grid Pattern

The main grid shows the user's active work:

- Filter to relevant records by default (current user's items, open status, team scope)
- Include sortable columns for the most important fields
- Make the primary identifier (e.g., case title) a clickable record link
- Show status with visual indicators (tags, colored text)
- Enable search and refresh
- Set a reasonable page size (10-20 rows)
- Include an "empty state" message when no records match

### Navigation Pattern

Provide navigation to other views:

- Link to "View All" for each record type grid
- Link to "Create New" for common record types
- Use `a!recordLink()` for record navigation
- Use `a!startProcessLink()` for action initiation

## Summary View Pattern

### Purpose
A summary view displays a single record's complete details. It answers: "What is this record?" and "What can I do with it?"

### Layout Structure

```
a!headerContentLayout()
├── header: Record title, status, key identifiers
└── contents:
    ├── Section: Primary Details
    │   └── a!columnsLayout() with label-value pairs
    ├── Section: [Domain-Specific Group] (1-3 sections)
    │   └── a!columnsLayout() with label-value pairs
    ├── Section: Related [Child Records]
    │   └── a!gridField() with related record data
    ├── Section: Related [Other Records] (if applicable)
    │   └── a!gridField() with related record data
    └── Section: Actions
        └── Action buttons or links
```

### Header Pattern

The header establishes context immediately:

- Record title as the primary heading (e.g., case title, customer name)
- Status displayed as a tag or badge with color coding
- Key identifiers visible (record ID, reference number)
- Created/modified dates if relevant

### Detail Section Pattern

Organize record fields into logical sections:

- **Primary Details** — the most important fields that identify the record
- **Domain-specific sections** — group by business meaning (e.g., "Contact Information", "Financial Details", "Assignment")
- Use two-column layouts with label-value pairs via `a!sideBySideLayout()`
- Labels in bold/strong style, values in normal style
- Use display components (rich text, tags) rather than read-only input fields
- Show empty fields as "—" or hide them with `showWhen`

### Related Data Pattern

Show child and related records in grids below the detail sections:

- One grid per related record type
- Include only the most relevant columns (3-5)
- Make identifiers clickable with record links
- Show record count in the section label (e.g., "Notes (5)")
- Include an "Add" link or button if the user can create related records

### Action Pattern

Surface available actions prominently:

- Use `a!buttonArrayLayout()` or a dedicated actions section
- Label buttons with the specific action ("Reassign", "Close", "Escalate") not generic labels ("Action", "Submit")
- Use appropriate button styles: PRIMARY for the main action, NORMAL for secondary, DESTRUCTIVE for delete/close
- Use `a!startProcessLink()` to trigger record action process models
- Show/hide actions based on record status and user permissions

## Form Pattern

### Purpose
Forms collect data for creating or updating records. A single form handles both modes using the `isUpdate` input.

### Layout Structure

```
a!formLayout()
├── contents:
│   ├── Section: [Primary Fields]
│   │   └── Required fields, key identifiers
│   ├── Section: [Domain Group 1]
│   │   └── Related fields grouped by meaning
│   ├── Section: [Domain Group 2] (if needed)
│   │   └── More grouped fields
│   └── Section: [Advanced/Optional] (collapsible)
│       └── Rarely-used fields
└── buttons: a!buttonLayout()
    ├── primaryButtons: Submit
    └── secondaryButtons: Cancel
```

### Field Organization Rules

1. **Group by business meaning** — "Contact Info", "Case Details", "Assignment" — not by data type
2. **Required fields first** within each section
3. **Related fields adjacent** — first name next to last name, start date next to end date
4. **Controlling fields before dependent fields** — a dropdown that shows/hides other fields comes before those fields
5. **Two-column layout** for forms with 6+ fields to reduce scrolling
6. **Collapsible sections** for advanced or optional field groups

### Create vs. Update Mode

Use the `isUpdate` input to control form behavior:

- **Section label**: `if(ri!isUpdate, "Edit Case", "New Case")`
- **Default values**: Initialize fields with defaults for create mode, use existing values for update mode
- **Read-only fields**: Lock certain fields in update mode (e.g., record type, created date)
- **Conditional sections**: Show update-only sections (e.g., "Resolution Details") only when `ri!isUpdate` is true

### Button Pattern

- **Submit button**: `style: "PRIMARY"`, `submit: true`, `validate: true`
- **Cancel button**: `style: "NORMAL"`, `submit: true`, `validate: false`, saves `true` into `ri!cancel`
- **Delete button** (if applicable): `style: "DESTRUCTIVE"`, with confirmation dialog
- Label the submit button with the action: "Create Case", "Save Changes", "Submit Request"

## Record Action Form Pattern

### Purpose
Record action forms handle specific operations on existing records — reassign, close, escalate, approve, reject. They're used inside process models triggered from record views.

### Layout Structure

```
a!formLayout()
├── contents:
│   ├── Section: Record Context (read-only)
│   │   └── Key fields from the record being acted on
│   ├── Section: [Action Fields]
│   │   └── Fields specific to this action
│   └── Section: [Comments/Notes] (optional)
│       └── Free-text field for action justification
└── buttons: a!buttonLayout()
    ├── primaryButtons: [Action-Specific Label]
    └── secondaryButtons: Cancel
```

### Context Display

Show enough record detail for the user to confirm they're acting on the right record:

- Record title/name as a heading or prominent display field
- Current status
- Key identifiers (ID, reference number)
- All context fields are read-only — the user is here to perform an action, not edit the record

### Action-Specific Fields

Keep action forms focused — only include fields needed for this specific action:

- **Reassign**: new assignee picker, optional reason
- **Close/Resolve**: resolution type dropdown, resolution notes
- **Escalate**: escalation reason, priority override
- **Approve/Reject**: decision dropdown or radio buttons, comments

### Button Labels

Use action-specific labels, not generic ones:

| Action | Primary Button Label | Style |
|---|---|---|
| Reassign | "Reassign" | PRIMARY |
| Close | "Close Case" | PRIMARY |
| Escalate | "Escalate" | PRIMARY |
| Approve | "Approve" | PRIMARY |
| Reject | "Reject" | DESTRUCTIVE |
| Delete | "Delete" | DESTRUCTIVE (with confirmation) |

## Section Organization Patterns

### Grouping by Business Domain

Organize fields into sections that match how users think about the data:

| Section Name | Typical Fields |
|---|---|
| General Information | Title, description, type, category |
| Contact Information | Name, email, phone, address |
| Assignment | Assigned to, team, department |
| Dates & Timeline | Start date, due date, completion date |
| Financial Details | Amount, currency, payment terms |
| Status & Resolution | Status, priority, resolution type, resolution notes |
| Additional Details | Notes, tags, custom fields |

### Section Sizing Guidelines

- **1-3 fields**: Single column, no section needed (or lightweight section)
- **4-8 fields**: Two-column layout within a section
- **9-15 fields**: Multiple sections with two-column layouts
- **15+ fields**: Consider a wizard or collapsible sections

### Collapsible Sections

Use collapsible sections for:
- Advanced configuration options
- Audit/history information
- Fields that are only relevant in certain states
- Optional metadata

Set `isCollapsible: true` and `isInitiallyCollapsed: true` for sections that most users won't need to see.

## Field Grouping Conventions

### Address Fields
```
Section: "Address"
├── Street Address (full width)
├── City | State/Province (two columns)
└── Postal Code | Country (two columns)
```

### Name Fields
```
Section: "Contact"
├── First Name | Last Name (two columns)
├── Email (full width)
└── Phone | Extension (two columns)
```

### Date Range Fields
```
├── Start Date | End Date (two columns, same row)
```

### Money Fields
```
├── Amount | Currency (two columns, same row)
```

## Responsive Design Considerations

- Two-column layouts stack to single column on mobile automatically (default `stackWhen: "PHONE"`)
- Keep critical information in the first column — it's always visible
- Don't use more than 3 columns for form fields — they become too narrow on tablets
- Test that collapsible sections work well on mobile (they should — Appian handles this)
- Grid columns should prioritize the most important fields — less important columns can be hidden on narrow screens with `showWhen` based on viewport (though this is rarely needed in practice)
