# Array Manipulation Patterns for Expression Rules

This file documents array operations, property access, and data derivation patterns for expression rules.

**Related references:**
- [function-reference.md](function-reference.md) - Function signatures and quick examples
- [null-safety-patterns.md](null-safety-patterns.md) - Null handling patterns
- [short-circuit-patterns.md](short-circuit-patterns.md) - Safe conditional evaluation

---

## Accessing Properties Across Arrays - Dot Notation

**The ONLY way to access a property across all items in an array of maps is using dot notation.**

### Correct: Dot Notation

```sail
local!items: {
  a!map(id: 1, name: "Item A", price: 100),
  a!map(id: 2, name: "Item B", price: 200),
  a!map(id: 3, name: "Item C", price: 300)
},

/* Access a single property across all items */
local!allNames: local!items.name,
/* Returns: {"Item A", "Item B", "Item C"} */

local!allPrices: local!items.price,
/* Returns: {100, 200, 300} */

/* Use in calculations */
local!totalPrice: sum(local!items.price),
/* Returns: 600 */

/* Use in comparisons */
local!contractTypes: intersection(local!items.type, {"Contract"}),
/* Returns all "Contract" values found */
```

### Wrong: property() Function Does NOT Exist

```sail
/* ERROR - property() is not a valid SAIL function */
local!allNames: property(local!items, "name", {}),

/* ERROR - This syntax is invalid */
sum(property(local!items, "price"))
```

### Key Rules

- Use `array.propertyName` to extract property values across all items
- Works with any array of maps (local variables with mock data)
- Returns an array of the property values in the same order
- Returns empty array `{}` if source array is empty
- Returns `null` elements for items missing that property

---

## Accessing Properties on Individual Map Items

**When working with individual map items (in a!forEach, single map, etc.), both dot notation and index() work.**

### Prefer Dot Notation (Simpler)

```sail
/* ✅ PREFERRED - Dot notation is simpler and more readable */
a!forEach(
  items: ri!items,
  expression: fv!item.name
)

/* Single map access */
local!item: a!map(id: 1, name: "Item A"),
local!name: local!item.name
```

### Use index() When Needed (Dynamic Keys or Null Safety)

```sail
/* ✅ Use index() when property key is dynamic */
a!forEach(
  items: ri!items,
  expression: index(fv!item, ri!propertyName, null)
)

/* ✅ Use index() for explicit null fallback */
index(local!item, "name", "Unknown")

/* ✅ Use index() in nested lookups */
index(
  index(local!config, wherecontains("status", local!config.key), null),
  "icon",
  "default-icon"
)
```

**Rule of thumb:** Use dot notation when the property name is known and fixed. Use index() when the property is dynamic or you need explicit null handling.

---

## Deriving Full Data from ID Arrays

**Common Pattern:** You have an array of IDs and need to derive full objects from those IDs.

**Solution:** Use `a!forEach() + index() + wherecontains()` to derive full objects from ID array.

### The Pattern

```sail
/* Step 1: Source data (all available items) */
local!availableItems: {
  a!map(id: 1, name: "Item A", type: "Public", price: 100),
  a!map(id: 2, name: "Item B", type: "Contract", price: 200),
  a!map(id: 3, name: "Item C", type: "Public", price: 150)
},

/* Step 2: ID array (from selection, user input, etc.) */
local!selectedIds: {1, 3},

/* Step 3: Derive full data using forEach + index + wherecontains */
local!selectedItems: a!forEach(
  items: local!selectedIds,
  expression: index(
    local!availableItems,
    wherecontains(fv!item, local!availableItems.id),
    null
  )
),
/* Returns: {
  a!map(id: 1, name: "Item A", type: "Public", price: 100),
  a!map(id: 3, name: "Item C", type: "Public", price: 150)
} */

/* Step 4: Use derived data for business logic */
local!totalPrice: sum(local!selectedItems.price),
/* Returns: 250 */

local!hasContractType: length(
  intersection(local!selectedItems.type, {"Contract"})
) > 0
/* Returns: false (no Contract items selected) */
```

### How It Works

1. **a!forEach(items: local!selectedIds, ...)** - Iterate over each selected ID
2. **fv!item** - Current ID being processed (e.g., 1, then 3)
3. **wherecontains(fv!item, local!availableItems.id)** - Find position of this ID in source array
   - Searches `{1, 2, 3}` (all IDs) for `fv!item` (current ID)
   - Returns array of positions: `{1}` or `{3}`
4. **index(local!availableItems, positions, null)** - Get full object at that position
   - Returns the complete `a!map(id: ..., name: ..., type: ..., price: ...)`
5. **null** - Default value if ID not found (defensive programming)

---

## Finding a Single Matching Item by ID

**Common Pattern:** You have a single ID and need to find the matching item from an array to access its fields.

### Correct: Use index() + wherecontains() + dot notation

```sail
/* Pattern: Find one item by ID and extract a field */
local!selectedOrgId: 5,
local!organizations: {
  a!map(id: 3, name: "Org A", type: "Nonprofit"),
  a!map(id: 5, name: "Org B", type: "Corporation"),
  a!map(id: 7, name: "Org C", type: "Government")
},

/* Get the organization type for the selected organization */
local!orgType: a!defaultValue(
  index(
    local!organizations,
    wherecontains(local!selectedOrgId, local!organizations.id),
    null
  ).type,  /* Use dot notation to extract the field */
  ""
)
/* Returns: "Corporation" */
```

### Wrong: Using a!forEach() creates array with nulls

```sail
/* DON'T DO THIS - Returns array like {null, "Corporation", null} */
local!orgType: a!defaultValue(
  index(
    a!forEach(
      items: local!organizations,
      expression: if(
        fv!item.id = local!selectedOrgId,
        fv!item.type,  /* Only ONE item matches */
        null  /* Everything else is null */
      )
    ),
    1,  /* ERROR: First element might be null! */
    null
  ),
  ""
)
```

**Why a!forEach() is wrong:**
- Creates an array where most elements are `null`
- `index(..., 1, null)` grabs the FIRST element, which could be `null`
- No guarantee the matching item is at position 1

### Key Rules

- Use `wherecontains(singleId, array.idField)` to find the position
- Use `index(array, position, null)` to get the matching item
- Use dot notation `.fieldName` to extract the field
- Wrap in `a!defaultValue()` to handle not-found cases

---

## Common Use Cases

### Use Case 1: Conditional Logic Based on Selection

```sail
local!selectedCourseIds: {2, 4},  /* From selection */
local!selectedCourses: a!forEach(
  items: local!selectedCourseIds,
  expression: index(
    local!availableCourses,
    wherecontains(fv!item, local!availableCourses.id),
    null
  )
),

/* Show field only if Contract course selected */
if(
  if(
    a!isNotNullOrEmpty(local!selectedCourses),
    length(intersection(local!selectedCourses.type, {"Contract"})) > 0,
    false
  ),
  /* Additional logic for Contract courses */,
  {}
)
```

### Use Case 2: Calculating Totals

```sail
local!cartItemIds: {5, 12, 8},
local!cartItems: a!forEach(
  items: local!cartItemIds,
  expression: index(
    local!productCatalog,
    wherecontains(fv!item, local!productCatalog.productId),
    null
  )
),

local!cartTotal: sum(local!cartItems.price),
local!taxableItems: length(
  wherecontains(true, local!cartItems.isTaxable)
)
```

### Use Case 3: Lookup Tables

```sail
/* Status to icon/color mapping */
local!statusId: 2,
local!statusConfig: {
  a!map(id: 1, status: "Open", icon: "folder-open", color: "#3B82F6"),
  a!map(id: 2, status: "In Progress", icon: "clock", color: "#F59E0B"),
  a!map(id: 3, status: "Completed", icon: "check-circle", color: "#10B981")
},

/* Get icon for current status */
local!icon: a!defaultValue(
  index(
    local!statusConfig,
    wherecontains(local!statusId, local!statusConfig.id),
    null
  ).icon,
  "file"
)
/* Returns: "clock" */
```

---

## Using wherecontains() Correctly

**Function Signature:** `wherecontains(valuesToFind, arrayToSearchIn)`
- **Returns:** Array of indices (1-based) where values are found
- **Always returns an array**, even if only one match

```sail
/* WRONG - wherecontains() only takes 2 parameters */
icon: wherecontains(value, statusArray, iconArray)  /* INVALID - 3 params */

/* RIGHT - Use nested index() for lookups */
local!statusConfig: {
  a!map(status: "Open", icon: "folder-open", color: "#059669"),
  a!map(status: "Closed", icon: "check-circle", color: "#6B7280"),
  a!map(status: "Pending", icon: "clock", color: "#F59E0B")
},

/* Extract matching config */
local!icon: index(
  index(
    local!statusConfig,
    wherecontains("Open", local!statusConfig.status),
    {}
  ).icon,
  1,
  "file"
)

/* How it works:
1. wherecontains("Open", local!statusConfig.status) -> {1}
2. index(local!statusConfig, {1}, {}) -> {a!map(status: "Open", icon: "folder-open", ...)}
3. .icon -> {"folder-open"}
4. index(..., 1, "file") -> "folder-open"
*/
```

### Common Pattern for Lookups

```sail
/* Find value from parallel arrays */
local!statuses: {"Open", "Closed", "Pending"},
local!colors: {"#059669", "#6B7280", "#F59E0B"},

local!color: index(
  local!colors,
  wherecontains("Open", local!statuses),
  "#000000"  /* Default color */
)
/* Returns: "#059669" (first element of colors array) */
```

---

## Array Type Initialization

**Problem:** In Appian, `{}` returns a **List of Variant** (untyped array), which causes type mismatch errors.

**Impact:** Functions like `contains()`, `wherecontains()`, `union()`, `intersection()`, and `append()` require typed arrays.

### ✅ ALWAYS Initialize Arrays with Type Declaration

```sail
/* ❌ WRONG - Untyped arrays cause errors */
local!selectedIds: {},
local!names: {},
local!dates: {},
local!flags: {},

/* ✅ CORRECT - Type-initialized arrays */
local!selectedIds: tointeger({}),        /* List of Integer */
local!names: touniformstring({}),        /* List of Text */
local!dates: todate({}),                 /* List of Date */
local!flags: toboolean({}),              /* List of Boolean */
```

### Complete Type Initialization Reference

| Function | Returns | Use Case |
|----------|---------|----------|
| `tointeger({})` | Empty List of Integer | IDs, counts, numeric values |
| `touniformstring({})` | Empty List of Text | Names, labels, text values |
| `toboolean({})` | Empty List of Boolean | Flags, checkboxes, yes/no |
| `todate({})` | Empty List of Date | Date values (no time) |
| `todatetime({})` | Empty List of DateTime | Timestamps, date+time |
| `todecimal({})` | Empty List of Decimal | Currency, percentages, precise numbers |
| `totime({})` | Empty List of Time | Time values (no date) |
| `touser({})` | Empty List of User | User references |
| `togroup({})` | Empty List of Group | Group references |

### 🚨 CRITICAL: tostring() vs touniformstring()

**DO NOT use `tostring({})` for array initialization!**

```sail
/* ❌ WRONG - tostring() merges array to single text string */
tostring({1, 2})        /* Returns: "1; 2" (one string, NOT an array!) */
tostring({})            /* Returns: "" (empty string, NOT empty array!) */

/* ✅ CORRECT - touniformstring() preserves array structure */
touniformstring({1, 2}) /* Returns: {"1", "2"} (array of strings) */
touniformstring({})     /* Returns: Empty List of Text */
```

### When to Use Each

- **`tostring()`** - Converts array TO a single merged string
  - Use when you want to concatenate array values into one string
  - Example: Display multiple values as comma-separated text

- **`touniformstring()`** - Converts array elements TO strings (preserves array)
  - Use for initializing text arrays
  - Use when you need an array of text values

---

## Array Modification Functions Reference

| Function | Purpose | Example |
|----------|---------|---------|
| `append(array, value)` | Add to end | `append({1,2}, 3)` → `{1,2,3}` |
| `insert(array, value, index)` | Insert at position | `insert({1,3}, 2, 2)` → `{1,2,3}` |
| `remove(array, index)` | Remove at position | `remove({1,2,3}, 2)` → `{1,3}` |
| `a!update(data, index, value)` | Replace at position | `a!update({1,2,3}, 2, 5)` → `{1,5,3}` |
| `wherecontains(val, array)` | Find positions | `wherecontains(2, {1,2,3})` → `{2}` |
| `index(array, pos, default)` | Get at position | `index({1,2,3}, 2, 0)` → `2` |

---

## Common Error Scenarios & Solutions

### Error 1: contains() with Untyped Array

**❌ WRONG:**
```sail
local!selectedCourseIds: {},

/* Later in code... */
contains(local!selectedCourseIds, fv!item.id)  /* ERROR: Variant vs Integer */
```

**✅ CORRECT:**
```sail
local!selectedCourseIds: tointeger({}),

/* Later in code... */
contains(local!selectedCourseIds, fv!item.id)  /* ✓ Both Integer type */
```

### Error 2: wherecontains() with Untyped Array

**❌ WRONG:**
```sail
local!filterValues: {},

wherecontains(local!filterValues, local!dataArray)  /* ERROR: Type mismatch */
```

**✅ CORRECT:**
```sail
local!filterValues: touniformstring({}),

wherecontains(local!filterValues, local!dataArray)  /* ✓ Typed array */
```

### Error 3: Mixed-Type Append Creates List of Variant

**❌ WRONG:**
```sail
local!items: {},

/* Appending different types creates List of Variant */
a!save(local!items, append(local!items, "text"))
a!save(local!items, append(local!items, 123))  /* Now List of Variant! */
```

**✅ CORRECT:**
```sail
/* Use consistent types with type initialization */
local!items: touniformstring({}),
a!save(local!items, append(local!items, "text"))
a!save(local!items, append(local!items, tostring(123)))  /* Convert to text */
```

---

## When Type Initialization is NOT Needed

### Arrays Initialized with Data

```sail
/* ✅ Type inferred from source data */
local!courseIds: local!availableCourses.id,        /* Already List of Integer */
local!courseNames: local!availableCourses.name,    /* Already List of Text */

/* ✅ Type inferred from literal array */
local!statuses: {"Open", "In Progress", "Closed"}, /* Already List of Text */
local!counts: {0, 5, 10, 15},                      /* Already List of Integer */
```

### Arrays That Will Be Fully Replaced

```sail
/* ✅ Acceptable if never used before replacement */
local!tempData: {},

/* Immediately replaced with typed data */
local!tempData: local!queryResult.data
```

---

## Summary

**🚨 MANDATORY RULE:**
> **ALWAYS initialize empty arrays with type-casting functions when storing primitive types (Integer, Text, Boolean, Date, DateTime, Decimal, Time, User, Group).**

**Key Patterns:**
1. **Dot notation** - Use `array.propertyName` to access properties across all items (property() does NOT exist)
2. **Derive from IDs** - Use `a!forEach() + index() + wherecontains()` pattern
3. **Single item lookup** - Use `index(array, wherecontains(id, array.id), null).field`
4. **Type initialization** - Use `tointeger({})`, `touniformstring({})`, etc. for empty arrays
5. **Critical distinction** - Use `touniformstring({})` for text arrays, NOT `tostring({})`
6. **wherecontains()** - Takes exactly 2 parameters, returns array of indices (1-based)
