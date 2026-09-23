# Record Type User Filters

User filters let end users filter the record list via dropdowns or date pickers. They appear at the top of the record list and are configured on the record type itself.

## When to Add User Filters

Add user filters when requirements mention:
- "filtering capabilities" or "search and filter"
- "users should be able to filter by..."
- "sortable/filterable record list"

User filters are step 10 in the dependency order — add them after record type fields and relationships exist, since `sourceRef` requires a field UUID.

## Filter Types

| facetType | Use for | How it works |
|---|---|---|
| `LIST_OF_VALUES` | Text, status, category, user fields | Dropdown of distinct values from the field |
| `DATE_RANGE` | Date or datetime fields | From/to date picker |
| `EXPRESSION` | Custom bucketing, computed filters | SAIL expression defines options and filter logic |

---

## LIST_OF_VALUES

Creates a dropdown populated with distinct values from the referenced field.

### Schema

```json
{
  "name": "Status",
  "facetType": "LIST_OF_VALUES",
  "sourceRef": "<field-uuid>"
}
```

### Optional Properties

| Property | Purpose |
|---|---|
| `allowMultipleSelections` | `true` (default) allows selecting multiple options |
| `useRelatedRecordValues` | `true` when sourceRef points to a related record field — auto-populates options from the related record |
| `relatedRecordDisplayField` | UUID of a Text field on the related RT to use as option labels (required with `useRelatedRecordValues`) |
| `relatedRecordSort` | `ASCENDING`, `DESCENDING`, or `UNSORTED` (required with `useRelatedRecordValues`) |

### Examples

**Simple list filter on a direct field:**
```json
{
  "name": "Status",
  "facetType": "LIST_OF_VALUES",
  "sourceRef": "<status-field-uuid>"
}
```

**Filter on a related record field (e.g., Organization name via relationship):**
```json
{
  "name": "Organization",
  "facetType": "LIST_OF_VALUES",
  "sourceRef": "<relationship-uuid>/<org-name-field-uuid>",
  "useRelatedRecordValues": true,
  "relatedRecordDisplayField": "<org-name-field-uuid>",
  "relatedRecordSort": "ASCENDING"
}
```

### Related Record Filters

When filtering by a field on a related record type, use the `<relationship-uuid>/<field-uuid>` format for `sourceRef`:

```json
{
  "name": "Department",
  "facetType": "LIST_OF_VALUES",
  "sourceRef": "<relationship-uuid>/<related-field-uuid>",
  "useRelatedRecordValues": true,
  "relatedRecordDisplayField": "<text-field-uuid-on-related-rt>",
  "relatedRecordSort": "ASCENDING"
}
```

This shows options from the related record (e.g., department names) rather than raw foreign key IDs.

---

## DATE_RANGE

Creates a from/to date picker that filters records within a date window.

### Schema

```json
{
  "name": "Submitted Date",
  "facetType": "DATE_RANGE",
  "sourceRef": "<field-uuid>"
}
```

### Optional Properties

| Property | Purpose |
|---|---|
| `startDateExpression` | Default "from" date when the record list loads (SAIL expression) |
| `endDateExpression` | Default "to" date when the record list loads (SAIL expression) |

### Examples

**Date range filter on a datetime field:**
```json
{
  "name": "Created Date",
  "facetType": "DATE_RANGE",
  "sourceRef": "<created-at-field-uuid>"
}
```

**With default date range (last 30 days):**
```json
{
  "name": "Submitted Date",
  "facetType": "DATE_RANGE",
  "sourceRef": "<field-uuid>",
  "startDateExpression": "today() - 30",
  "endDateExpression": "today()"
}
```

---

## EXPRESSION

Fully custom filter using SAIL. You define both the options shown to users and the filter logic applied when selected. Use for custom bucketing, computed values, or filters that don't map to a single field.

### Schema

```json
{
  "name": "Priority Level",
  "facetType": "EXPRESSION",
  "expression": "a!recordFilterList(name: \"Priority Level\", options: {...})"
}
```

The `expression` must evaluate to either:
- `a!recordFilterList()` — for list-style options
- `a!recordFilterDateRange()` — for expression-driven date range

### a!recordFilterList Pattern

```
a!recordFilterList(
  name: "Label shown to users",
  options: {
    a!recordFilterListOption(
      id: 1,
      name: "Option Label",
      filter: a!queryFilter(
        field: 'recordType!{uuid}Name.fields.{fieldUuid}fieldName',
        operator: "=",
        value: "some value"
      )
    ),
    a!recordFilterListOption(
      id: 2,
      name: "Another Option",
      filter: a!queryFilter(
        field: 'recordType!{uuid}Name.fields.{fieldUuid}fieldName',
        operator: ">",
        value: 100
      )
    )
  }
)
```

### Examples

**Status bucket filter with custom grouping:**
```json
{
  "name": "Status Group",
  "facetType": "EXPRESSION",
  "expression": "a!recordFilterList(name: \"Status\", options: {a!recordFilterListOption(id: 1, name: \"Open\", filter: a!queryFilter(field: 'recordType!{rt-uuid}Submission.fields.{fid}status', operator: \"in\", value: {\"New\", \"In Progress\", \"Under Review\"})), a!recordFilterListOption(id: 2, name: \"Closed\", filter: a!queryFilter(field: 'recordType!{rt-uuid}Submission.fields.{fid}status', operator: \"in\", value: {\"Approved\", \"Rejected\", \"Cancelled\"}))})"
}
```

**Numeric range buckets (e.g., salary bands):**
```json
{
  "name": "Amount Range",
  "facetType": "EXPRESSION",
  "expression": "a!recordFilterList(name: \"Amount\", options: {a!recordFilterListOption(id: 1, name: \"Under $1,000\", filter: a!queryFilter(field: 'recordType!{rt-uuid}Order.fields.{fid}amount', operator: \"<\", value: 1000)), a!recordFilterListOption(id: 2, name: \"$1,000 - $10,000\", filter: a!queryFilter(field: 'recordType!{rt-uuid}Order.fields.{fid}amount', operator: \"between\", value: {1000, 10000})), a!recordFilterListOption(id: 3, name: \"Over $10,000\", filter: a!queryFilter(field: 'recordType!{rt-uuid}Order.fields.{fid}amount', operator: \">\", value: 10000))})"
}
```

**Boolean field as friendly labels:**
```json
{
  "name": "Active Status",
  "facetType": "EXPRESSION",
  "expression": "a!recordFilterList(name: \"Active\", options: {a!recordFilterListOption(id: 1, name: \"Active\", filter: a!queryFilter(field: 'recordType!{rt-uuid}Employee.fields.{fid}isActive', operator: \"=\", value: true)), a!recordFilterListOption(id: 2, name: \"Inactive\", filter: a!queryFilter(field: 'recordType!{rt-uuid}Employee.fields.{fid}isActive', operator: \"=\", value: false))})"
}
```

### When to Use EXPRESSION vs LIST_OF_VALUES

| Scenario | Use |
|---|---|
| Filter by distinct values of a field (status, category) | `LIST_OF_VALUES` |
| Filter by date window | `DATE_RANGE` |
| Group values into buckets (salary ranges, age bands) | `EXPRESSION` |
| Boolean field with friendly labels ("Active"/"Inactive") | `EXPRESSION` |
| Combine multiple fields in one filter option | `EXPRESSION` |
| Filter by computed value or expression | `EXPRESSION` |

Prefer `LIST_OF_VALUES` and `DATE_RANGE` when possible — they're simpler, auto-maintain their options, and require no SAIL. Use `EXPRESSION` only when the guided types can't express the filter logic.

---

## Common Workflow

1. **Get field UUIDs** — list the record type's fields to find the UUIDs you need for `sourceRef`
2. **Add filters based on field types:**
   - Text/status fields → `LIST_OF_VALUES`
   - Date/datetime fields → `DATE_RANGE`
   - Custom bucketing → `EXPRESSION`
3. **Verify** — list filters on the record type to confirm they were created correctly

## Design Guidance

- Create one filter per meaningful filterable dimension (status, category, date, assignee)
- Use `LIST_OF_VALUES` for fields with a bounded set of distinct values
- Use `DATE_RANGE` for date/datetime fields
- Use `EXPRESSION` for custom bucketing or multi-field logic
- Filter `name` should be a user-friendly label (e.g., "Status", "Submitted Date")
- Add filters after relationships are established if filtering on related fields
- For foreign key fields that reference a lookup table, use `useRelatedRecordValues: true` to show human-readable labels instead of IDs
