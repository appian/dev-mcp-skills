# Short-Circuit Evaluation Patterns

This file documents safe conditional evaluation patterns for expression rules. These patterns prevent crashes caused by SAIL's non-short-circuit behavior in `and()` and `or()` functions.

**Related references:**
- [function-reference.md](function-reference.md) - Function signatures and quick examples
- [null-safety-patterns.md](null-safety-patterns.md) - Null handling patterns
- [array-patterns.md](array-patterns.md) - Array manipulation and property access

---

## The Problem

**SAIL's `and()` and `or()` functions DO NOT short-circuit** - they evaluate ALL arguments even if the result is already determined.

```sail
/* ❌ ERROR - and() evaluates BOTH arguments */
/* If local!computedData is empty, the second argument still evaluates */
/* This causes: "Invalid index: Cannot index property 'type' of Null" */
and(
  a!isNotNullOrEmpty(local!computedData),
  local!computedData.type = "Contract"  /* CRASHES if computedData is empty */
)
```

**Why this is critical:** When accessing properties on potentially empty variables, the property access will execute even if the null check returns false, causing a runtime error.

---

## The Solution: Nested if()

```sail
/* ✅ if() short-circuits - only evaluates the returned branch */
if(
  if(
    a!isNotNullOrEmpty(local!computedData),
    local!computedData.type = "Contract",  /* Only evaluated when not empty */
    false
  ),
  /* Then branch - show field */,
  /* Else branch - hide field */
)
```

**How it works:**
1. Outer `if()` receives the result of the inner `if()`
2. Inner `if()` checks `a!isNotNullOrEmpty(local!computedData)`
3. If true, evaluates `local!computedData.type = "Contract"` safely
4. If false, returns `false` without accessing `.type` property
5. Outer `if()` uses the result to determine which branch to execute

---

## When to Use Nested if() vs and()

### Use nested if() when:
- Checking null/empty before property access on computed variables
- Any scenario where the second condition CANNOT be safely evaluated if the first is false
- Accessing properties on variables that could be empty arrays or null
- Comparing values that require the first condition to be true

### Use and() when:
- All conditions are independent and can be safely evaluated in any order
- All variables involved are guaranteed to have values (not null, not empty)
- Simple boolean combinations without property access

---

## Quick Reference Table

| Function | Short-Circuits? | Use For |
|----------|----------------|---------|
| `if()` | ✅ Yes - Only evaluates returned branch | Null-safe property access, conditional logic, binary conditions |
| `and()` | ❌ No - Evaluates all arguments | Independent boolean conditions only (never for null safety) |
| `or()` | ❌ No - Evaluates all arguments | Independent boolean conditions only (never for null safety) |
| `a!match()` | ✅ Yes - Only evaluates matched branch | Pattern matching - single value against 3+ options |

---

## Common Scenarios Requiring Nested if()

### Scenario 1: Computed Variables from Selections

```sail
/* Selection derives full data */
local!selectedItems: a!forEach(
  items: local!selectedIds,
  expression: index(...)
),

/* ❌ WRONG - Crashes when no items selected */
showWhen: and(
  length(local!selectedItems) > 0,
  local!selectedItems.type = "Contract"  /* ERROR if empty */
)

/* ✅ RIGHT - Nested if() prevents crash */
showWhen: if(
  if(
    a!isNotNullOrEmpty(local!selectedItems),
    length(intersection(local!selectedItems.type, {"Contract"})) > 0,
    false
  ),
  true,
  false
)
```

### Scenario 2: Array Property Access

```sail
/* ❌ WRONG - Crashes if items array is empty */
and(
  length(local!items) > 0,
  local!items.price > 100  /* ERROR if items is {} */
)

/* ✅ RIGHT */
if(
  if(
    a!isNotNullOrEmpty(local!items),
    length(where(local!items.price > 100)) > 0,
    false
  ),
  /* Then branch */,
  /* Else branch */
)
```

### Scenario 3: Filtered or Derived Arrays

```sail
local!activeUsers: a!forEach(
  items: local!allUsers,
  expression: if(fv!item.status = "Active", fv!item, null)
),

/* ✅ Checking properties with nested if() */
if(
  if(
    a!isNotNullOrEmpty(local!activeUsers),
    wherecontains("ADMIN", local!activeUsers.role),
    false
  ),
  /* Show admin panel */,
  {}
)
```

### Scenario 4: Multiple Property Checks

```sail
/* Checking multiple properties on same variable */
if(
  if(
    a!isNotNullOrEmpty(local!selectedItem),
    and(
      local!selectedItem.type = "Contract",
      local!selectedItem.status = "Active"
    ),
    false
  ),
  /* Both properties checked safely */,
  {}
)
```

---

## The Pattern for Null-Safe Property Access

**Always use nested if() pattern when accessing properties on computed variables:**

```sail
if(
  if(
    a!isNotNullOrEmpty(local!variable),
    /* Safe to access properties here - variable is guaranteed not empty */
    local!variable.propertyName = "expectedValue",
    /* Return safe default - false, null, or {} depending on context */
    false
  ),
  /* Then branch - condition is true */,
  /* Else branch - condition is false or variable is null */
)
```

**Template breakdown:**
- **Outer if()** - Controls which branch to execute based on the result
- **Inner if()** - Performs the null check and conditional property access
- **Null check** - `a!isNotNullOrEmpty(local!variable)`
- **Property access** - Only executed when variable is not empty
- **Default return** - `false` (or appropriate default for context)

---

## Common Mistakes

### ❌ MISTAKE 1: Using and() for Null-Safe Property Access

```sail
/* WRONG - and() doesn't short-circuit */
and(
  a!isNotNullOrEmpty(local!data),
  local!data.field = "value"  /* Executes even if data is empty! */
)
```

**Fix:** Use nested if() instead (see pattern above)

### ❌ MISTAKE 2: Forgetting the Default Return

```sail
/* WRONG - Missing default in inner if() */
if(
  if(
    a!isNotNullOrEmpty(local!variable),
    local!variable.type = "Contract"
    /* Missing: , false */
  ),
  ...
)
```

**Fix:** Always provide the default return value: `false`, `null`, or `{}`

### ❌ MISTAKE 3: Using or() for Null Checks

```sail
/* WRONG - or() doesn't short-circuit either */
or(
  a!isNullOrEmpty(local!data),
  local!data.field <> "value"  /* Executes even if data is empty! */
)
```

**Fix:** Use nested if() with appropriate logic reversal

---

## Advanced Patterns

### Combining Multiple Conditions

```sail
/* Check multiple variables with nested if() */
if(
  if(
    and(
      a!isNotNullOrEmpty(local!items),
      a!isNotNullOrEmpty(local!filters)
    ),
    and(
      local!items.status = "Active",
      wherecontains(local!filters.type, local!items.type)
    ),
    false
  ),
  /* Both checks pass */,
  {}
)
```

### Using with a!match()

```sail
/* a!match() also short-circuits (safe to use) */
a!match(
  value: local!variable,
  equals: null, then: "No data",
  equals: "Contract", then: "Show contract fields",
  default: "Show standard fields"
)
```

---

## Summary

**🚨 CRITICAL RULE:**
> **Never use and() or or() for null-safe property access. Always use nested if() when the second condition depends on the first being true.**

**Key Patterns:**
1. **Computed variables** - Always use nested if() before accessing properties
2. **Array property access** - Use nested if() when array could be empty
3. **Filtered data** - Use nested if() for derived arrays
4. **Multiple properties** - Outer if() for null check, inner and() for property checks
5. **Default returns** - Always provide `false`, `null`, or `{}` as appropriate

**Remember:**
- `if()` ✅ Short-circuits (safe)
- `a!match()` ✅ Short-circuits (safe)
- `and()` ❌ Does NOT short-circuit (unsafe for null checks)
- `or()` ❌ Does NOT short-circuit (unsafe for null checks)
