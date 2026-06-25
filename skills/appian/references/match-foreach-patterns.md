# Pattern Matching and Iteration Patterns

This file documents `a!match()` for pattern matching and `a!forEach()` for iteration, including the complete fv! variable reference.

**Related references:**
- [function-reference.md](function-reference.md) - Function signatures and quick examples
- [null-safety-patterns.md](null-safety-patterns.md) - Null handling patterns
- [short-circuit-patterns.md](short-circuit-patterns.md) - Safe conditional evaluation

---

## a!match() Pattern Matching

### ⚠️ CRITICAL: equals/whenTrue Are SCALAR Values

**The `equals` and `whenTrue` parameters accept SCALAR values ONLY, NOT arrays.**

```sail
/* ❌ COMMON MISTAKE - Using arrays */
a!match(
  value: local!specialty,
  equals: {1, 4, 2},  /* INVALID SYNTAX! */
  then: {
    {...},  /* First result */
    {...},  /* Second result */
    {...}   /* Third result */
  },
  default: {}
)

/* ✅ CORRECT PATTERN - Multiple equals/then pairs */
a!match(
  value: local!specialty,
  equals: 1, then: {...},  /* First condition and result */
  equals: 4, then: {...},  /* Second condition and result */
  equals: 2, then: {...},  /* Third condition and result */
  default: {}
)
```

**Key principle:** Each `equals` matches ONE value and returns ONE result via `then`. For multiple conditions, chain multiple `equals, then` pairs.

---

## When to Use a!match() vs if()

### Use a!match() for:
- ✅ Single value compared against multiple options (3+ cases)
- ✅ Status, category, priority, type mappings
- ✅ Cleaner and more maintainable than nested `if()` statements
- ✅ Short-circuits like `if()` - safe for conditional logic

### Use if() for:
- Multiple different variables in the condition
- Complex boolean expressions (not just equality checks)
- Binary decisions (only 2 possible outcomes)

---

## Pattern: Status to Icon/Color Mapping

```sail
/* ❌ OLD PATTERN - Nested if() (hard to read) */
backgroundColor: if(
  statusCode = "INTEGRATED",
  "POSITIVE",
  if(
    or(statusCode = "SUBMITTED", statusCode = "VALIDATED"),
    "ACCENT",
    if(
      statusCode = "DRAFT",
      "#F59E0B",
      "SECONDARY"
    )
  )
)

/* ✅ NEW PATTERN - a!match() (clean, maintainable) */
backgroundColor: a!match(
  value: statusCode,
  equals: "INTEGRATED", then: "POSITIVE",
  equals: "SUBMITTED", then: "ACCENT",
  equals: "VALIDATED", then: "ACCENT",
  equals: "DRAFT", then: "#F59E0B",
  default: "SECONDARY"
)
```

**Benefits:**
- Flat structure - no nesting hell
- Clear value→result mapping
- Easy to add/remove cases
- Self-documenting code
- Less error-prone

---

## Pattern: equals for Exact Value Matching

### Use `equals` for:
- ✅ Exact value matching: status codes, categories, types
- ✅ Enumerated values: "Open", "Closed", "Pending"
- ✅ String/ID comparisons: Known discrete values

```sail
/* Simple status-based styling */
a!match(
  value: local!status,
  equals: "Active", then: "#059669",
  equals: "Pending", then: "#D97706",
  equals: "Inactive", then: "#6B7280",
  default: "#000000"
)

/* Icon mapping */
a!match(
  value: local!priority,
  equals: "Critical", then: "exclamation-triangle",
  equals: "High", then: "arrow-up",
  equals: "Medium", then: "minus",
  equals: "Low", then: "arrow-down",
  default: "info-circle"
)
```

---

## Pattern: whenTrue for Range-Based Comparisons

### Use `whenTrue` for:
- ✅ Numeric ranges: scores, percentages, amounts, counts
- ✅ Date/time intervals: days elapsed, time since event
- ✅ Threshold logic: >=, <=, between X and Y
- ✅ Any comparison operators: <, >, <=, >=, <>

```sail
/* ❌ AVOID - Nested if() for ranges */
backgroundColor: if(
  fv!row.performance >= 110,
  "#059669",
  if(
    fv!row.performance >= 90,
    "#FCD34D",
    "#DC2626"
  )
)

/* ✅ PREFER - a!match() with whenTrue */
backgroundColor: a!match(
  value: fv!row.performance,
  whenTrue: fv!value >= 110, then: "#059669",
  whenTrue: fv!value >= 90, then: "#FCD34D",
  default: "#DC2626"
)
```

### Pattern: Date Elapsed Ranges

```sail
/* Display message based on days since case creation */
a!match(
  value: tointeger(today() - local!caseCreatedOn),
  whenTrue: fv!value <= 30, then: "Less than 30 days",
  whenTrue: fv!value <= 60, then: "30-60 days",
  whenTrue: fv!value <= 90, then: "60-90 days",
  default: "More than 90 days"
)
```

### Pattern: Percentage Thresholds (Heatmaps, KPIs)

```sail
/* Color-code performance percentage */
a!match(
  value: local!percentComplete,
  whenTrue: fv!value >= 100, then: "POSITIVE",
  whenTrue: fv!value >= 75, then: "ACCENT",
  whenTrue: fv!value >= 50, then: "#F59E0B",
  default: "NEGATIVE"
)
```

### Pattern: Currency/Amount Tiers

```sail
/* Assign tier based on order amount */
a!match(
  value: local!orderTotal,
  whenTrue: fv!value >= 10000, then: "Platinum",
  whenTrue: fv!value >= 5000, then: "Gold",
  whenTrue: fv!value >= 1000, then: "Silver",
  default: "Bronze"
)
```

### Pattern: Mixed equals + whenTrue

```sail
/* Combine equals (exact matches) and whenTrue (ranges) */
a!match(
  value: local!itemCount,
  equals: 0, then: "Empty cart",
  equals: 1, then: "1 item in cart",
  whenTrue: fv!value > 1, then: fv!value & " items in cart",
  default: "Unknown"
)
```

### Key Rules for whenTrue

1. **`fv!value`** references the `value` parameter in `whenTrue`, `then`, and `default`
2. **Order matters** - conditions evaluate top-to-bottom, first match wins
3. **No upper bounds needed** when using descending order (>= 110, >= 90, default)
4. **Keywords required** - always use `equals:`, `whenTrue:`, `then:`, `default:`
5. **Can mix patterns** - combine `equals` and `whenTrue` in same expression

---

## ✅ Best Practice: PREFER a!match() Over Nested if()

**When you have a single value to compare against 3+ options, ALWAYS use `a!match()` instead of nested `if()` statements.**

### When to Keep if()

- Multiple different variables in the condition
- Complex boolean expressions (not just equality checks)
- Computed logic that can't be reduced to pattern matching

**Example - Multiple Variables (Use if()):**
```sail
showWhen: if(
  and(local!userRole = "Manager", local!department = "Sales"),
  local!salesAmount > 10000,
  false
)
```

This checks TWO variables with AND logic - can't use `a!match()`.

---

## Short-Circuit Behavior

`a!match()` short-circuits like `if()` - it only evaluates the matched branch:

| Function | Short-Circuits? | Use For |
|----------|----------------|---------|
| `if()` | ✅ Yes - Only evaluates returned branch | Null-safe property access, conditional logic, binary conditions |
| `and()` | ❌ No - Evaluates all arguments | Independent boolean conditions only (never for null safety) |
| `or()` | ❌ No - Evaluates all arguments | Independent boolean conditions only (never for null safety) |
| `a!match()` | ✅ Yes - Only evaluates matched branch | Pattern matching - single value against 3+ options |

**For complete short-circuit patterns, see [short-circuit-patterns.md](short-circuit-patterns.md).**

---

## a!forEach() Iteration

### Available Function Variables

When using `a!forEach()`, Appian automatically provides these function variables within the `expression` parameter:

| Variable | Type | Description | Common Use Cases |
|----------|------|-------------|------------------|
| `fv!item` | Any | Current item value from the array | Accessing properties, creating UI components, data transformations |
| `fv!index` | Integer | Current iteration position (1-based) | Array manipulation, numbering, position-based logic |
| `fv!isFirst` | Boolean | `true` only on first iteration | Special formatting for first item, conditional headers |
| `fv!isLast` | Boolean | `true` only on last iteration | Special formatting for last item, conditional footers |
| `fv!itemCount` | Integer | Total number of items in array | Progress indicators, conditional logic based on total |

**⚠️ CRITICAL:** These variables are **ONLY** available inside `a!forEach()` expressions. They do NOT exist in other contexts.

---

## fv!item - Accessing Current Item

**Purpose:** Access the current item's value or properties during iteration

**SAIL-Specific Syntax:**
- **Map properties:** Use dot notation: `fv!item.title`
- **Scalar values:** Use `fv!item` directly

```sail
/* Map data with dot notation */
a!forEach(
  items: {
    a!map(title: "Overview", icon: "info-circle"),
    a!map(title: "Details", icon: "list")
  },
  expression: a!map(
    label: fv!item.title,  /* Dot notation for maps */
    icon: fv!item.icon
  )
)
```

---

## fv!index - Current Position (1-Based)

**Purpose:** Get the current iteration position for array manipulation or position-based logic

**IMPORTANT:** Appian uses 1-based indexing. First item is index 1, not 0.

### Common Pattern: Removing Items from Arrays

```sail
/* Remove button for dynamic lists */
a!forEach(
  items: local!items,
  expression: a!cardLayout(
    contents: {
      a!textField(
        label: "Item " & fv!index,  /* Use for labeling */
        value: fv!item.text,
        saveInto: fv!item.text
      )
    },
    link: a!dynamicLink(
      label: "Remove",
      value: fv!index,  /* Pass index to identify which item to remove */
      saveInto: {
        a!save(
          local!items,
          remove(
            local!items,
            fv!index  /* Remove at this position */
          )
        )
      }
    )
  )
)
```

### Parallel Array Lookups (Status/Icon Mapping)

```sail
/* Map colors and icons to status values by position */
local!statuses: {"Open", "In Progress", "Completed"},
local!icons: {"folder-open", "clock", "check-circle"},
local!colors: {"#EF4444", "#F59E0B", "#10B981"},

a!forEach(
  items: local!statuses,
  expression: a!stampField(
    icon: index(local!icons, fv!index, "file"),      /* Match by position */
    backgroundColor: index(local!colors, fv!index, "#6B7280"),
    contentColor: "#FFFFFF",
    size: "MEDIUM"
  )
)
```

---

## fv!isFirst - First Iteration Detection

**Purpose:** Detect the first iteration for conditional headers or skipping separators

```sail
a!forEach(
  items: local!sections,
  expression: {
    /* Add divider before all items EXCEPT the first */
    if(
      fv!isFirst,
      {},  /* No divider for first item */
      a!columnsLayout(
        columns: {a!columnLayout(contents: {}, width: "FILL")},
        marginBelow: "STANDARD"
      )
    ),
    /* Then show the content */
    a!cardLayout(
      contents: {
        a!richTextDisplayField(value: fv!item.content)
      }
    )
  }
)
```

---

## fv!isLast - Last Iteration Detection

**Purpose:** Detect the last iteration for conditional footers, "Add" buttons, or skipping separators

```sail
/* Add button or total only after last item */
a!forEach(
  items: local!invoiceItems,
  expression: {
    /* Show each line item */
    a!columnsLayout(
      columns: {
        a!columnLayout(
          contents: {a!richTextDisplayField(value: fv!item.description)}
        ),
        a!columnLayout(
          contents: {a!richTextDisplayField(value: "$" & fv!item.amount)},
          width: "NARROW"
        )
      }
    ),
    /* Show total only after last item */
    if(
      fv!isLast,
      a!richTextDisplayField(
        value: a!richTextItem(
          text: "Total: $" & sum(local!invoiceItems.amount),
          size: "LARGE",
          style: "STRONG"
        )
      ),
      {}
    )
  }
)
```

---

## fv!itemCount - Total Item Count

**Purpose:** Access the total number of items for progress indicators or conditional logic

```sail
/* Progress indicator pattern */
a!forEach(
  items: local!uploadQueue,
  expression: a!cardLayout(
    contents: {
      a!richTextDisplayField(
        value: a!richTextItem(
          text: "Processing " & fv!index & " of " & fv!itemCount,
          style: "STRONG"
        )
      ),
      a!textField(
        label: "File",
        value: fv!item.filename,
        readOnly: true
      )
    }
  )
)
```

---

## Combining Multiple Variables

```sail
/* Dynamic remove button logic with fv!itemCount and fv!isLast */
a!forEach(
  items: local!lineItems,
  expression: a!cardLayout(
    contents: {
      a!textField(
        label: "Item " & fv!index,
        value: fv!item.name
      )
    },
    link: if(
      and(
        fv!itemCount > 1,  /* Only show remove if more than 1 item */
        not(fv!isLast)     /* Keep at least the last item */
      ),
      a!dynamicLink(
        label: "Remove",
        value: fv!index,
        saveInto: a!save(local!lineItems, remove(local!lineItems, fv!index))
      ),
      {}
    )
  )
)
```

---

## Common Mistakes

### ❌ MISTAKE 1: Accessing properties on scalar values

```sail
/* ❌ WRONG - fv!item is a string, not an object */
a!forEach(
  items: {"Open", "Closed"},
  expression: fv!item.status  /* ❌ ERROR */
)

/* ✅ RIGHT */
a!forEach(
  items: {"Open", "Closed"},
  expression: a!tagField(text: fv!item)  /* ✅ fv!item IS the value */
)
```

### ❌ MISTAKE 2: Using 0-based indexing

```sail
/* ❌ WRONG - Appian is 1-based */
a!forEach(
  items: local!items,
  expression: if(fv!index = 0, ...)  /* First item is 1, not 0! */
)

/* ✅ RIGHT */
a!forEach(
  items: local!items,
  expression: if(fv!index = 1, ...)  /* First item is 1 */
)
```

### ❌ MISTAKE 3: Using fv! variables outside a!forEach()

```sail
/* ❌ WRONG - fv! variables don't exist outside forEach */
local!displayValue: fv!item.name  /* ERROR: fv! not in scope */

/* ✅ RIGHT - Use fv! only inside forEach expression */
a!forEach(
  items: local!items,
  expression: fv!item.name  /* ✓ Works inside forEach */
)
```

---

## Complete fv! Variable Reference Table

| Variable | Type | Available In | Description | Example Use |
|----------|------|--------------|-------------|-------------|
| `fv!item` | Any | `a!forEach()` | Current iteration value | `fv!item.name`, `fv!item` |
| `fv!index` | Integer | `a!forEach()` | Position (1-based) | `remove(array, fv!index)` |
| `fv!isFirst` | Boolean | `a!forEach()` | True on first iteration | Skip first divider |
| `fv!isLast` | Boolean | `a!forEach()` | True on last iteration | Show total after last item |
| `fv!itemCount` | Integer | `a!forEach()` | Total items in array | "Processing 3 of 10" |
| `fv!value` | Any | `a!match()` whenTrue/then/default | Value being matched | `whenTrue: fv!value >= 90` |

**Note:** Grid components (`a!gridField`) use `fv!row` and `fv!currentPage`, NOT the a!forEach() variables.

---

## Summary

**🚨 MANDATORY RULES:**
1. **a!match() equals/whenTrue are SCALAR** - Use multiple equals/then pairs, NOT arrays
2. **PREFER a!match() over nested if()** - For single value against 3+ options
3. **Use equals for exact matches** - Status, category, type comparisons
4. **Use whenTrue for ranges** - Numeric thresholds, date intervals, any comparison operators
5. **fv! variables ONLY in a!forEach()** - Don't use outside forEach context
6. **Appian is 1-based** - First item is index 1, not 0
7. **fv!index for array manipulation** - Use with remove(), insert(), a!update()
8. **fv!isFirst/isLast for conditional rendering** - Headers, footers, dividers
9. **a!match() and if() short-circuit** - Safe for conditional logic (unlike and/or)

**Key Patterns:**
- **Status mapping** - Use a!match() with equals
- **Range comparisons** - Use a!match() with whenTrue
- **Remove buttons** - Use fv!index with remove()
- **Progress indicators** - Use fv!index and fv!itemCount
- **Conditional dividers** - Use fv!isFirst/isLast
