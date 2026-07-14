# Dropdown Patterns

**Purpose:** Patterns for populating choice fields from lookup table record types vs hardcoded values.

**When to use:** Building interfaces with choice fields for selection (Priority, Status, Type, Category, etc.).

**Applies to all choice components:** `a!dropdownField`, `a!multipleDropdownField`, `a!checkboxField`, and `a!radioButtonField`. All four accept a records-powered `data` parameter and all four require `choiceLabels` and `choiceValues`. Examples below use `a!dropdownField`; the same rules apply to the others.

> ⚠️ **`choiceLabels` and `choiceValues` are ALWAYS required — even when you pass a record type to `data`.** The `data` parameter replaces the *query*, not the choice mappings. A choice field with `data` but no `choiceValues`/`choiceLabels` renders empty. See "Two ways to populate from a record type" below.

---

## Decision Tree: Query vs Hardcode

```
Need a dropdown field?
├─ Is this a lookup table record type (Priority, Status, Type, Category)?
│  │
│  ├─ YES → Query from record type (Section 1)
│  │  - Values can change without code changes
│  │  - Respects isActive flag
│  │  - Admin-managed data
│  │  - Examples: CM Case Priority, CM Case Status, Order Type, Ticket Category
│  │
│  └─ NO → Is it a fixed set of values (never changes)?
│     │
│     ├─ YES → Hardcode is acceptable (Section 2)
│     │  - True/False, Yes/No
│     │  - Days of week, Months
│     │  - System constants
│     │
│     └─ NO → Query from record type (Section 1)
```

---

## Section 1: Populate from Lookup Table Record Type

**Use when:**
- Lookup table record type exists (Priority, Status, Type, Category)
- Values can be added/removed/modified by admins
- Values have isActive flag for soft delete

### Two ways to populate from a record type

| Approach | How | When to prefer |
|----------|-----|----------------|
| **`data` parameter** (simplest) | Pass the record type (or `a!recordData()`) to `data`; point `choiceValues`/`choiceLabels` at record fields | Straightforward "all active rows" dropdowns — no local variable needed |
| **Query into local + `index()`** | `a!queryRecordType(...).data` into a `local!`, then map with `index()` | When you reuse the same data elsewhere, need complex filtering, or must transform values |

**Both require `choiceLabels` AND `choiceValues`.** Neither approach lets you omit them.

### Pattern: `data` Parameter (Preferred for Simple Lookups)

Pass the record type to `data` and point the choice parameters at record fields — `choiceValues` at the primary key, `choiceLabels` at the display field. No local variable or `index()` needed.

```sail
a!dropdownField(
  label: "Priority",
  placeholder: "Select a priority",
  data: a!recordData(
    recordType: 'recordType!{uuid}CM Case Priority',
    filters: a!queryFilter(
      field: 'recordType!{uuid}CM Case Priority.fields.{uuid}isActive',
      operator: "=",
      value: true
    )
  ),
  /* choiceValues + choiceLabels are STILL REQUIRED with data */
  choiceValues: 'recordType!{uuid}CM Case Priority.fields.{uuid}id',
  choiceLabels: 'recordType!{uuid}CM Case Priority.fields.{uuid}label',
  /* sort the choices; field is the literal "choiceLabels" or "choiceValues" */
  sort: a!sortInfo(field: "choiceLabels", ascending: true),
  value: local!priorityId,
  saveInto: local!priorityId,
  required: true
)
```

**Notes:**
- Default behavior if you accept it: labels use the first text field, values use the primary key — but set them explicitly.
- To build a label from multiple fields, use `fv!data`: `choiceLabels: fv!data['recordType!{uuid}Location.fields.{uuid}city'] & " (" & fv!data['recordType!{uuid}Location.fields.{uuid}state'] & ")"`. You **cannot** reference related-record fields via `fv!data` in `choiceLabels`.
- `sort` uses the literal string `"choiceLabels"` or `"choiceValues"` as its `field`, not a record field reference.

### Pattern: Query Active Lookup Values (into a local variable)

❌ **WRONG - Hardcoded values:**
```sail
a!dropdownField(
  label: "Priority",
  choiceLabels: {"Low", "Medium", "High", "Critical"},
  choiceValues: {1, 2, 3, 4},
  value: local!priorityId,
  saveInto: local!priorityId
)
```

**Problems with hardcoding:**
- Admin adds new priority → Code must be updated
- Can't deactivate a priority without code changes
- Out of sync if priority IDs change
- Violates single source of truth

✅ **CORRECT - Query from record type:**
```sail
a!localVariables(
  /* Query active priorities once at interface load */
  local!priorities: a!queryRecordType(
    recordType: 'recordType!{uuid}CM Case Priority',
    fields: {
      'recordType!{uuid}CM Case Priority.fields.{uuid}id',
      'recordType!{uuid}CM Case Priority.fields.{uuid}label'
    },
    filters: a!queryFilter(
      field: 'recordType!{uuid}CM Case Priority.fields.{uuid}isActive',
      operator: "=",
      value: true
    ),
    pagingInfo: a!pagingInfo(
      startIndex: 1,
      batchSize: 100,
      sort: a!sortInfo(
        field: 'recordType!{uuid}CM Case Priority.fields.{uuid}sortOrder',
        ascending: true
      )
    )
  ).data,
  
  a!formLayout(
    contents: {
      a!dropdownField(
        label: "Priority",
        placeholder: "Select a priority",
        choiceLabels: index(
          local!priorities,
          'recordType!{uuid}CM Case Priority.fields.{uuid}label',
          {}
        ),
        choiceValues: index(
          local!priorities,
          'recordType!{uuid}CM Case Priority.fields.{uuid}id',
          {}
        ),
        value: local!priorityId,
        saveInto: local!priorityId,
        required: true
      )
    }
  )
)
```

**Benefits:**
- ✅ Values dynamically loaded from database
- ✅ Admins can add/remove priorities without code changes
- ✅ Respects isActive flag (only shows active options)
- ✅ Respects sortOrder for consistent display order
- ✅ Single source of truth (record type)

---

### Pattern: Multiple Dropdowns from Different Lookup Tables

```sail
a!localVariables(
  /* Query all lookup tables once */
  local!priorities: a!queryRecordType(
    recordType: 'recordType!{uuid}CM Case Priority',
    fields: {id, label},
    filters: a!queryFilter(field: isActive, operator: "=", value: true),
    pagingInfo: a!pagingInfo(1, 100, a!sortInfo(sortOrder, true))
  ).data,
  
  local!statuses: a!queryRecordType(
    recordType: 'recordType!{uuid}CM Case Status',
    fields: {id, label},
    filters: a!queryFilter(field: isActive, operator: "=", value: true),
    pagingInfo: a!pagingInfo(1, 100, a!sortInfo(sortOrder, true))
  ).data,
  
  a!formLayout(
    contents: {
      /* Priority dropdown */
      a!dropdownField(
        label: "Priority",
        choiceLabels: index(local!priorities, "label", {}),
        choiceValues: index(local!priorities, "id", {}),
        value: local!priorityId,
        saveInto: local!priorityId
      ),
      
      /* Status dropdown */
      a!dropdownField(
        label: "Status",
        choiceLabels: index(local!statuses, "label", {}),
        choiceValues: index(local!statuses, "id", {}),
        value: local!statusId,
        saveInto: local!statusId
      )
    }
  )
)
```

**Key points:**
- Query each lookup table ONCE in local variables (not inside dropdown)
- Use `index()` to extract labels and IDs
- Always filter by `isActive = true`
- Always sort by `sortOrder` field for consistent order

---

### Pattern: Displaying Selected Value (Read-Only Mode)

When showing the selected value in read-only mode, display the **label** not the ID:

❌ **WRONG - Shows ID:**
```sail
a!textField(
  label: "Priority",
  value: local!priorityId,  /* Shows "2" instead of "Medium" */
  readOnly: true
)
```

✅ **CORRECT - Shows label via relationship:**
```sail
/* If queried with relationship traversal */
a!textField(
  label: "Priority",
  value: local!case['recordType!{uuid}CM Case.relationships.{uuid}priority.fields.{uuid}label'],
  readOnly: true
)

/* Or lookup from priorities list */
a!textField(
  label: "Priority",
  value: if(
    a!isNotNullOrEmpty(local!priorityId),
    index(
      index(
        local!priorities,
        wherecontains(local!priorityId, index(local!priorities, "id", {})),
        {}
      ),
      "label",
      "Unknown"
    ),
    "N/A"
  ),
  readOnly: true
)
```

---

## Section 2: Hardcoded Values (When Acceptable)

**Use when:**
- Fixed set of values that NEVER change
- Not managed by admins
- No isActive flag needed

### Acceptable Use Cases:

**Boolean choices:**
```sail
a!dropdownField(
  label: "Is Active",
  choiceLabels: {"Yes", "No"},
  choiceValues: {true, false},
  value: local!isActive,
  saveInto: local!isActive
)
```

**System constants:**
```sail
a!dropdownField(
  label: "Day of Week",
  choiceLabels: {"Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"},
  choiceValues: {1, 2, 3, 4, 5, 6, 7},
  value: local!dayOfWeek,
  saveInto: local!dayOfWeek
)
```

**Fixed business rules:**
```sail
a!dropdownField(
  label: "Visibility",
  choiceLabels: {"Public", "Internal", "Private"},
  choiceValues: {"PUBLIC", "INTERNAL", "PRIVATE"},
  value: local!visibility,
  saveInto: local!visibility
)
```

---

## Section 3: Common Mistakes

### Mistake 0: Passing `data` without `choiceValues`/`choiceLabels`

The most common error. Passing a record type to `data` does **not** auto-populate the options — the field renders empty. `choiceValues` and `choiceLabels` are required regardless.

❌ **WRONG - `data` alone, no choice mappings:**
```sail
a!dropdownField(
  label: "Priority",
  data: 'recordType!{uuid}CM Case Priority'
  /* No choiceValues/choiceLabels → dropdown is empty */
)
```

✅ **CORRECT - `data` plus record-field choice mappings:**
```sail
a!dropdownField(
  label: "Priority",
  data: 'recordType!{uuid}CM Case Priority',
  choiceValues: 'recordType!{uuid}CM Case Priority.fields.{uuid}id',
  choiceLabels: 'recordType!{uuid}CM Case Priority.fields.{uuid}label',
  value: local!priorityId,
  saveInto: local!priorityId
)
```

This applies equally to `a!multipleDropdownField`, `a!checkboxField`, and `a!radioButtonField`.

---

### Mistake 1: Querying Inside Dropdown

❌ **WRONG - Query runs on every interaction:**
```sail
a!dropdownField(
  choiceLabels: index(
    a!queryRecordType(...).data,  /* Query runs repeatedly */
    "label",
    {}
  )
)
```

✅ **CORRECT - Query once in local variable:**
```sail
local!priorities: a!queryRecordType(...).data,

a!dropdownField(
  choiceLabels: index(local!priorities, "label", {})
)
```

---

### Mistake 2: Forgetting isActive Filter

❌ **WRONG - Shows inactive values:**
```sail
a!queryRecordType(
  recordType: 'recordType!Priority',
  fields: {id, label}
  /* No filter on isActive */
)
```

✅ **CORRECT - Filter to active only:**
```sail
a!queryRecordType(
  recordType: 'recordType!Priority',
  fields: {id, label},
  filters: a!queryFilter(
    field: 'recordType!Priority.fields.isActive',
    operator: "=",
    value: true
  )
)
```

---

### Mistake 3: Not Sorting by sortOrder

❌ **WRONG - Inconsistent order:**
```sail
a!queryRecordType(
  recordType: 'recordType!Priority',
  fields: {id, label},
  pagingInfo: a!pagingInfo(1, 100)
  /* No sort specified - order is random */
)
```

✅ **CORRECT - Sort by sortOrder field:**
```sail
a!queryRecordType(
  recordType: 'recordType!Priority',
  fields: {id, label},
  pagingInfo: a!pagingInfo(
    startIndex: 1,
    batchSize: 100,
    sort: a!sortInfo(
      field: 'recordType!Priority.fields.sortOrder',
      ascending: true
    )
  )
)
```

---

## Quick Decision Guide

| Scenario | Pattern |
|----------|---------|
| Priority, Status, Type, Category lookup tables | ✅ From record type (`data` param, or query into local) |
| Admin-managed values | ✅ From record type |
| Values can be added/removed | ✅ From record type |
| Simple "all active rows" dropdown | ✅ `data` parameter (simplest) |
| Reuse same data elsewhere / complex transforms | ✅ Query into local + `index()` |
| True/False, Yes/No | ✅ Hardcode acceptable |
| Days, Months, System constants | ✅ Hardcode acceptable |
| Fixed business rules (never change) | ✅ Hardcode acceptable |

**Whichever record-type approach you use, always set `choiceLabels` AND `choiceValues`.**

**When in doubt:** Populate from record type. It's more flexible and maintainable.

---

## Related Patterns

- **Field Type Mapping:** See `interfaces.md` for field type to component mapping
- **Query Patterns:** See `query-record-type-patterns.md` for advanced query examples
- **Null Safety:** See `null-safety-patterns.md` for handling empty dropdown results
