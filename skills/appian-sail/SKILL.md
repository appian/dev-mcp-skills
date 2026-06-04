---
name: "appian-sail"
description: "Write SAIL code for Appian interfaces — components, layouts, data binding, and interaction patterns. Covers the declarative UI component model, layout hierarchy, local variables, save behavior, conditional visibility, dynamic defaults, validation, grids, record links, and form submission with buttons. Use when building or modifying interface expressions, creating forms, dashboards, summary views, or any SAIL component tree."
---

## Interface Structure

Every SAIL interface is a single expression that returns a component tree. The expression starts with `=` and typically uses `a!localVariables()` to define state, then returns a top-level layout containing nested components.

Use `a!formLayout()` as the top-level layout for forms (start forms, task forms). Use `a!headerContentLayout()` for dashboards and landing pages. Use `a!wizardLayout()` for multi-step forms.

Top-level layouts cannot be nested inside other layouts.

### Standard Form Structure

```
a!localVariables(
  local!record: ri!record,
  local!isUpdate: ri!isUpdate,
  a!formLayout(
    contents: {
      a!sectionLayout(...)
    },
    buttons: a!buttonLayout(
      primaryButtons: { a!buttonWidget(...) },
      secondaryButtons: { a!buttonWidget(...) }
    )
  )
)
```

## Layout Hierarchy

Nest layouts to organize content. The standard nesting pattern is:

1. **Top-level layout** — `a!formLayout()`, `a!headerContentLayout()`, or `a!wizardLayout()`
2. **Sections** — `a!sectionLayout()` groups related fields under a heading
3. **Columns** — `a!columnsLayout()` with `a!columnLayout()` children arranges content side by side
4. **Components** — input fields, display fields, grids, buttons

Use `a!columnsLayout()` with two `a!columnLayout()` children for a standard two-column form layout. Use `a!sideBySideLayout()` with `a!sideBySideItem()` children for inline label-value pairs or compact arrangements.

Use `a!cardLayout()` to visually group content in a bordered or shaded container. Use `a!boxLayout()` for titled containers with a header bar.

### Section Layout

`a!sectionLayout()` takes `label`, `contents`, `showWhen`, and `validations` parameters. Use it to group related fields under a heading. Sections stack vertically inside the top-level layout's `contents` parameter.

### Columns Layout

`a!columnsLayout()` takes a `columns` parameter containing `a!columnLayout()` items. Each column takes `contents` and optionally `width` (for fixed or relative sizing). Columns stack on narrow screens by default.

```
a!columnsLayout(
  columns: {
    a!columnLayout(contents: { /* left column */ }),
    a!columnLayout(contents: { /* right column */ })
  }
)
```

## Data Binding

### Local Variables

Use `a!localVariables()` at the top of the interface expression to define mutable state. Local variables hold form data, UI state flags, and computed values.

```
a!localVariables(
  local!record: ri!record,
  local!showDetails: false,
  /* return a component tree here */
)
```

Local variables are mutable — update them with `a!save()` in component event parameters like `saveInto`.

### Interface Inputs

Interface inputs use the `ri!` prefix. They are passed in from the parent context (process model, site page, or another interface). The `ri!` name must exactly match the input name defined on the interface object.

For forms used as process model start forms, the standard inputs are:
- `ri!record` (or entity-specific name like `ri!case`) — the record data
- `ri!isUpdate` — Boolean controlling create vs. edit mode
- `ri!cancel` — Boolean for cancel flow

### Saving Values

Use `a!save()` in `saveInto` parameters to update local variables or rule inputs when users interact with components.

```
a!textField(
  label: "Name",
  value: local!record[recordType!Employee.fields.firstName],
  saveInto: a!save(
    local!record[recordType!Employee.fields.firstName],
    save!value
  )
)
```

`save!value` is a special variable available inside `saveInto` that contains the new value from user input.

For simple cases where the save target matches the value source, you can use the shorthand:

```
a!textField(
  label: "Name",
  value: local!name,
  saveInto: local!name
)
```

Multiple saves can be chained in a list:

```
saveInto: {
  a!save(local!record[recordType!Case.fields.status], "Open"),
  a!save(local!statusChanged, true)
}
```

## Interaction Patterns

### Conditional Visibility

Use `showWhen` on any component or layout to control visibility. The value must be a Boolean expression.

```
a!sectionLayout(
  label: "Update Details",
  showWhen: local!isUpdate,
  contents: { ... }
)
```

Common patterns:
- Show/hide sections based on mode: `showWhen: ri!isUpdate`
- Show/hide fields based on another field's value: `showWhen: local!status = "Other"`
- Role-based visibility: `showWhen: a!isUserMemberOfGroup(loggedInUser(), cons!ADMIN_GROUP)`

When `showWhen` is `false`, the component is not rendered and its expressions are not evaluated.

### Dynamic Defaults

Set default values for new records using conditional logic:

```
a!localVariables(
  local!record: if(
    ri!isUpdate,
    ri!record,
    recordType!Case(
      recordType!Case.fields.status: cons!STATUS_OPEN,
      recordType!Case.fields.createdDate: today()
    )
  ),
  /* form contents */
)
```

### Validation

Add field-level validation with the `validations` parameter. Return a list of text strings — empty list means valid, non-empty strings display as error messages.

```
a!textField(
  label: "Email",
  value: local!email,
  saveInto: local!email,
  required: true,
  requiredMessage: "Email is required",
  validations: if(
    a!isNullOrEmpty(local!email),
    {},
    if(
      find("@", local!email) > 0,
      {},
      "Enter a valid email address"
    )
  )
)
```

Use `required: true` for mandatory fields. Use `requiredMessage` to customize the required field message. Validations are evaluated on form submission by default.

Form-level validations go in the top-level layout's `validations` parameter and display at the top of the form.

### Read-Only Mode

Make fields read-only by setting `readOnly: true`. This displays the value without allowing edits. Use this for summary views or to lock fields based on status:

```
a!textField(
  label: "Status",
  value: local!record[recordType!Case.fields.status],
  readOnly: true
)
```

For entire read-only views, prefer display components (`a!richTextDisplayField`, `a!sideBySideLayout` with label-value pairs) over read-only input fields.

## Form Patterns

### Submit and Cancel Buttons

Place buttons in the `buttons` parameter of `a!formLayout()` using `a!buttonLayout()`:

```
a!buttonLayout(
  primaryButtons: {
    a!buttonWidget(
      label: "Submit",
      submit: true,
      style: "SOLID",
      color: "ACCENT",
      validationGroup: "submit"
    )
  },
  secondaryButtons: {
    a!buttonWidget(
      label: "Cancel",
      value: true,
      saveInto: ri!cancel,
      submit: true,
      style: "OUTLINE",
      validate: false
    )
  }
)
```

Key patterns:
- `submit: true` closes the form and returns control to the process model
- `validate: true` (default) triggers validation before submit; set to `false` for Cancel buttons
- `style` must be one of: `"SOLID"`, `"OUTLINE"`, `"GHOST"`, `"LINK"` — `"PRIMARY"`, `"SECONDARY"`, `"NORMAL"` are invalid and will fail
- `style: "SOLID"` + `color: "ACCENT"` for the primary action; `"OUTLINE"` for secondary/cancel; `"SOLID"` + `color: "NEGATIVE"` for destructive actions
- Cancel buttons save `true` into `ri!cancel` so the process model can skip record writes

### Create/Update Form Pattern

Use `ri!isUpdate` to control form behavior for consolidated create/update forms:

```
a!formLayout(
  contents: {
    a!sectionLayout(
      label: if(ri!isUpdate, "Edit Case", "New Case"),
      contents: {
        a!textField(
          label: "Title",
          value: local!record[recordType!Case.fields.title],
          saveInto: a!save(local!record[recordType!Case.fields.title], save!value),
          required: true
        ),
        a!dropdownField(
          label: "Priority",
          placeholder: "Select a priority",
          choiceLabels: { "Low", "Medium", "High" },
          choiceValues: { "Low", "Medium", "High" },
          value: local!record[recordType!Case.fields.priority],
          saveInto: a!save(local!record[recordType!Case.fields.priority], save!value),
          required: true
        )
      }
    )
  },
  buttons: a!buttonLayout(...)
)
```

## Grid Patterns

### Record Grid (Read-Only)

Use `a!gridField()` with `a!recordData()` to display a grid of records:

```
a!gridField(
  label: "Open Cases",
  data: a!recordData(
    recordType: recordType!Case,
    filters: a!queryLogicalExpression(
      operator: "AND",
      filters: {
        a!queryFilter(
          field: recordType!Case.fields.status,
          operator: "=",
          value: "Open"
        )
      }
    )
  ),
  columns: {
    a!gridColumn(
      label: "Title",
      sortField: recordType!Case.fields.title,
      value: fv!row[recordType!Case.fields.title]
    ),
    a!gridColumn(
      label: "Status",
      sortField: recordType!Case.fields.status,
      value: fv!row[recordType!Case.fields.status]
    ),
    a!gridColumn(
      label: "Assigned To",
      value: fv!row[recordType!Case.relationships.assignedUser][recordType!User.fields.name]
    )
  },
  pageSize: 10,
  showSearchBox: true,
  showRefreshButton: true
)
```

Inside grid column `value` expressions, use `fv!row` to access the current row's record data.

### Record Link in Grid

Make grid values clickable to navigate to a record view. **Do NOT put `a!recordLink()` directly in `a!gridColumn(value:)` or wrap it in `a!linkField()`** — both serialize to raw `@attributes` text at runtime.

CORRECT pattern — wrap in `a!richTextDisplayField`:

```
a!gridColumn(
  label: "Name",
  value: a!richTextDisplayField(
    label: "name",
    labelPosition: "COLLAPSED",
    value: a!richTextItem(
      text: fv!row['recordType!{RT_UUID}Name.fields.{FIELD_UUID}fieldName'],
      link: a!recordLink(
        recordType: 'recordType!{RT_UUID}Name',
        identifier: fv!row['recordType!{RT_UUID}Name.fields.{PK_UUID}id']
      )
    )
  )
)
```

### Editable Grid

Use `a!gridLayout()` for editable grids where users add, edit, and remove rows:

```
a!gridLayout(
  label: "Line Items",
  headerCells: {
    a!gridLayoutHeaderCell(label: "Item"),
    a!gridLayoutHeaderCell(label: "Quantity"),
    a!gridLayoutHeaderCell(label: "Actions")
  },
  rows: a!forEach(
    items: local!lineItems,
    expression: a!gridRowLayout(
      contents: {
        a!textField(
          value: fv!item.name,
          saveInto: a!save(local!lineItems[fv!index].name, save!value)
        ),
        a!integerField(
          value: fv!item.quantity,
          saveInto: a!save(local!lineItems[fv!index].quantity, save!value)
        ),
        a!richTextDisplayField(
          value: a!richTextItem(
            text: "Remove",
            link: a!dynamicLink(
              saveInto: a!save(local!lineItems, remove(local!lineItems, fv!index))
            )
          )
        )
      }
    )
  ),
  addRowLink: a!dynamicLink(
    label: "Add Item",
    saveInto: a!save(
      local!lineItems,
      append(local!lineItems, a!map(name: "", quantity: 0))
    )
  )
)
```

## Link Types

Use the appropriate link type for navigation and actions:

- `a!recordLink()` — navigate to a record view (requires `recordType` and `identifier`)
- `a!dynamicLink()` — trigger a `saveInto` action without navigation (for in-page interactions)
- `a!startProcessLink()` — start a process model (requires `processModel`)
- `a!safeLink()` — open an external URL
- `a!authorizationLink()` — link to an authorization page
- `a!documentDownloadLink()` — download a document
- `a!userRecordLink()` — link to a user's record

Wrap links in `a!linkField()` for standalone display or use them inside `a!richTextDisplayField()` with `a!richTextItem()`.

## Iterating in Interfaces

Use `a!forEach()` to render a list of components dynamically. Inside the expression, use `fv!item`, `fv!index`, `fv!isFirst`, `fv!isLast`.

```
a!forEach(
  items: local!items,
  expression: a!cardLayout(
    contents: {
      a!richTextDisplayField(
        value: a!richTextItem(text: fv!item.name, style: "STRONG")
      )
    },
    showWhen: true
  )
)
```

## Dropdown Patterns

### Static Dropdown

```
a!dropdownField(
  label: "Status",
  placeholder: "Select a status",
  choiceLabels: { "Open", "In Progress", "Closed" },
  choiceValues: { "Open", "In Progress", "Closed" },
  value: local!status,
  saveInto: local!status
)
```

### Record-Backed Dropdown

For dropdowns populated from a reference table:

```
a!localVariables(
  local!statuses: a!queryRecordType(
    recordType: recordType!CaseStatus,
    fields: {
      recordType!CaseStatus.fields.id,
      recordType!CaseStatus.fields.name
    },
    pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100)
  ).data,
  a!dropdownField(
    label: "Status",
    placeholder: "Select a status",
    choiceLabels: local!statuses[recordType!CaseStatus.fields.name],
    choiceValues: local!statuses[recordType!CaseStatus.fields.id],
    value: local!record[recordType!Case.fields.statusId],
    saveInto: a!save(local!record[recordType!Case.fields.statusId], save!value)
  )
)
```

## Pitfalls

- Using plain-text record type names in SAIL expressions submitted via the API — all `recordType!` references must use the UUID-qualified format wrapped in single quotes: `'recordType!{rtUuid}Name.fields.{fieldUuid}fieldName'`. Plain-text names fail at parse time. See the Appian Expressions skill for the full format specification.
- Nesting a top-level layout (`a!formLayout`, `a!headerContentLayout`) inside another layout — top-level layouts must be the outermost component
- Forgetting `save!value` in `a!save()` — without it, the saved value is null
- Using `a!save()` outside of a `saveInto` parameter — `a!save()` only works inside `saveInto`
- Setting `validate: true` on Cancel buttons — this prevents cancellation when required fields are empty
- Forgetting `submit: true` on form buttons — without it, the button saves values but doesn't close the form
- Using `a!gridField()` without `a!recordData()` — `a!gridField` expects a record data source for record-backed grids
- Referencing `fv!row` outside of a grid column's `value` expression — `fv!row` is only available inside grid columns
- Referencing `fv!item` outside of `a!forEach()` — function variables are scoped to their parent function
- Putting `a!columnLayout()` outside of `a!columnsLayout()` — columns must be direct children of a columns layout
- Using `a!forEach()` to render components but forgetting it returns a list — the parent parameter must accept a list

## When You Need More

For questions about specific SAIL components, advanced patterns, or component parameters beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.

### Reference Files

Load these when you need authoritative detail on a specific topic:

**Component catalog** — `references/component-reference.md`
Full parameter tables for layout, input, display, grid, and action components.

**Per-component instructions** — `references/components/`
Detailed usage rules for individual components (one file per component):
- `button-instructions.md`, `card-choice-field-instructions.md`, `chart-instructions.md`
- `grid-field-instructions.md`, `grid-layout-instructions.md`, `image-field-instructions.md`
- `rich-text-instructions.md`, `stamp-field-instructions.md`, `tabular-data-display-pattern.md`

**Layout instructions** — `references/layouts/`
Detailed usage rules for layout components:
- `card-layout-instructions.md`, `columns-layout-instructions.md`, `form-layout-instructions.md`
- `header-content-layout-instructions.md`, `pane-layout-instructions.md`, `sidebyside-layout-instructions.md`
- `tab-layout-instructions.md`, `wizard-layout-instructions.md`

**Design patterns** — `references/patterns/`
Reusable UI patterns with SAIL examples:
- `card_lists.md`, `kpis.md`, `messages.md`, `tabs.md`

**Icon aliases** — `references/rich-text-icon-aliases.md`
Full list of icon names available for `a!richTextIcon()` and `a!stampField()`.

## Platform Quirks

Version-specific SAIL knowledge that prevents common validation and runtime errors.

### 1. Record Type References

Always use the full quoted UUID syntax: `'recordType!{uuid}Name'`. The shorthand `recordType!Name` does NOT work in expression validation. For field paths: `'recordType!{rt-uuid}Name.fields.{field-uuid}fieldName'`.

### 2. Button Styles

Valid `style` values: `"SOLID"`, `"OUTLINE"`, `"GHOST"`, `"LINK"`. Do NOT use `"PRIMARY"`, `"NORMAL"`, or `"SECONDARY"` — these cause validation errors.

### 3. Grid Column Links

NEVER put `a!recordLink()` directly in `a!gridColumn(value:)`. It serializes to raw `@attributes` text at runtime. Always wrap in `a!richTextDisplayField` (see "Record Link in Grid" above).

### 4. Grid Filtering

On `a!gridField` with record-type data, use parameter name `queryFilter` (not `filters`). The `filters` parameter passes syntax validation but is silently ignored at runtime. Note: `filters` IS correct on `a!queryRecordType`, NOT on `a!gridField`.

### 5. Record Count Queries

Use `fetchTotalCount: true` on `a!queryRecordType` to get accurate counts via `.totalCount`:

```
a!queryRecordType(
  recordType: 'recordType!{UUID}Name',
  filters: a!queryFilter(field: 'recordType!{UUID}Name.fields.{FIELD_UUID}field', operator: "=", value: targetValue),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 1),
  fetchTotalCount: true
).totalCount
```

### 6. Grid Selection Style

Do NOT use `selectionStyle: "NONE"` — not a valid value. Valid values: `"CHECKBOX"`, `"ROW_HIGHLIGHT"`, `"CHECKBOX_SUBTLE_HIGHLIGHT"`, `"SUBTLE_HIGHLIGHT"`. To disable selection, omit the parameter entirely.

### 7. Site Page Visibility

### 7. Site Page Visibility

Standard visibility expressions work as expected. Use `"true"` or `"=true"` for always-visible pages. For conditional visibility, use expressions like `a!isUserMemberOfGroup(loggedInUser(), cons!GROUP_NAME)`.

### 8. headerContentLayout Header Slot

The `header` parameter of `a!headerContentLayout` requires `a!cardLayout` or `a!billboardLayout`. `a!sectionLayout` is rejected in the header slot.

### 9. Process Model Node Types

Standard gateway type IDs: XOR = `core.4`, AND = `core.2`, OR = `core.3`, Start = `core.0`, End = `core.1`.

### 10. Query Syntax

`a!queryRecordType` with shorthand `recordType!Name` fails in expression validation. Use the quoted full-path form: `'recordType!{uuid}Name'`. Sort uses: `a!sortInfo(field: ..., ascending: true())`.
