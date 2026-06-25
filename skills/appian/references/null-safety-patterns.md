# Null Safety Patterns for Expression Rules

This file documents null handling patterns specifically for expression rules. These patterns prevent runtime errors caused by null or empty values.

**Related references:**
- [function-reference.md](function-reference.md) - Function signatures and quick examples
- [short-circuit-patterns.md](short-circuit-patterns.md) - Safe conditional evaluation patterns
- [array-patterns.md](array-patterns.md) - Array manipulation and property access

---

## CHECKPOINT

**Before finalizing any expression, verify EVERY direct field or variable reference uses appropriate null safety:**

### For Variables:
- ✅ `a!defaultValue(local!variable, "")`
- ✅ `a!defaultValue(local!variable, null)`
- ✅ `a!defaultValue(local!array, {})`
- ❌ `local!variable` (naked variable reference without null safety)

### For Record Fields (when used in interfaces):
- ✅ `a!defaultValue(ri!record['recordType!Example.fields.field'], "")`
- ✅ `a!defaultValue(ri!record['recordType!Example.fields.field'], null)`
- ✅ `a!defaultValue(ri!record['recordType!Example.relationships.rel'], {})`
- ❌ `ri!record['recordType!Example.fields.field']` (naked field reference)

---

## Functions That Reject Null

**Some functions fail even with `a!defaultValue()` and require `if()` checks BEFORE calling:**

### Null-Rejecting Functions:

| Function Category | Functions | Why They Fail |
|-------------------|-----------|---------------|
| **User/Group** | `user(userId, property)`, `group(groupId, property)` | Cannot accept null ID |
| **Formatting** | `text(value, format)` | Cannot format null dates/numbers |
| **String manipulation** | `upper()`, `lower()`, `left()`, `right()`, `find()` | Fail on null input |
| **Logical** | `not()` | Cannot accept null value |

### Required Pattern:

```sail
/* ✅ CORRECT - Check for null BEFORE calling function */
if(
  a!isNotNullOrEmpty(a!defaultValue(fieldValue, null)),
  functionThatRejectsNull(fieldValue, otherParams),
  fallbackValue
)

/* ❌ WRONG - a!defaultValue() wrapper doesn't prevent the error */
functionThatRejectsNull(a!defaultValue(fieldValue, null), otherParams)
```

**Rule**: When a function operates ON a value (transforms/formats it), check for null BEFORE calling. The `a!defaultValue()` wrapper alone is insufficient.

---

## Critical Patterns by Function

### 1. user() Function - CRITICAL Pattern

**The user() function CANNOT accept null as the first parameter. Check for null BEFORE calling:**

```sail
/* ✅ CORRECT - Check for null BEFORE calling user() */
if(
  a!isNotNullOrEmpty(a!defaultValue(userIdField, null)),
  trim(
    user(userIdField, "firstName") & " " & user(userIdField, "lastName")
  ),
  "–"
)

/* ❌ WRONG - Checking null INSIDE user() call - user() will fail if passed null */
trim(
  user(a!defaultValue(userIdField, null), "firstName") & " " &
  user(a!defaultValue(userIdField, null), "lastName")
)
```

**Critical Note**: The user() function CANNOT accept null as the first parameter. If the field value is null, user() will cause an error. You MUST check for null with an if() statement BEFORE calling user(), not inside the user() function call.

**Note on User Display Names:**
- Use `user(userId, "firstName") & " " & user(userId, "lastName")` instead of `displayName`
- The `displayName` field is actually a nickname and is not always populated
- Wrap in `trim()` to clean up any extra whitespace
- Use "–" (en dash) as fallback for null/empty users instead of text like "Unknown User" or "Unassigned"

### 2. not() Function - CRITICAL Pattern

**The `not()` function cannot accept null. When using `not()` with variables or rule inputs that might be null, use `a!defaultValue()` to provide a fallback:**

```sail
/* ❌ WRONG - Direct use of not() with potentially null value */
readOnly: not(ri!isEditable)  /* Fails if ri!isEditable is null */
disabled: not(local!allowEdits)  /* Fails if local!allowEdits is null */

/* ✅ CORRECT - Use a!defaultValue() to provide fallback */
readOnly: not(a!defaultValue(ri!isEditable, false()))  /* Returns true if null */
disabled: not(a!defaultValue(local!allowEdits, false()))  /* Returns true if null */

/* ✅ ALTERNATIVE - Use if() to check for null first */
readOnly: if(
  a!isNullOrEmpty(ri!isEditable),
  true(),  /* Default to read-only if null */
  not(ri!isEditable)
)
```

### Common scenarios requiring null protection:
- `readOnly: not(ri!isEditable)` → Use `not(a!defaultValue(ri!isEditable, false()))`
- `disabled: not(local!allowEdits)` → Use `not(a!defaultValue(local!allowEdits, false()))`
- `showWhen: not(local!isHidden)` → Use `not(a!defaultValue(local!isHidden, false()))`

**Best Practice**: Always wrap rule inputs and variables in `a!defaultValue()` before passing to `not()`. Choose the default value (`true()` or `false()`) based on the desired behavior when the value is null.

### 3. text() Function with Date/DateTime Values

**The text() function CANNOT accept null values. Always check for null before calling text():**

```sail
/* ❌ WRONG - Passing null to text() causes errors */
text(a!defaultValue(local!dateField, null), "MMM d, yyyy")

/* ✅ CORRECT - Check for null BEFORE calling text() */
if(
  a!isNullOrEmpty(a!defaultValue(local!dateField, null)),
  "–",
  text(local!dateField, "MMM d, yyyy")
)

/* ✅ CORRECT - Alternative pattern with defaultValue as fallback string */
if(
  a!isNullOrEmpty(a!defaultValue(local!dueDate, null)),
  "No due date",
  text(local!dueDate, "MM/DD/YYYY")
)
```

**Rule**: When formatting dates with text(), ALWAYS wrap in a null check that returns a fallback string (like "–" or "N/A"), NOT null.

### 4. String Manipulation Functions

String functions like `upper()`, `lower()`, `left()`, `right()`, and `find()` fail on null input:

```sail
/* ❌ WRONG - Function fails if value is null */
upper(local!status)

/* ✅ CORRECT - Check for null first */
if(
  a!isNotNullOrEmpty(local!status),
  upper(local!status),
  ""
)

/* ✅ ALTERNATIVE - Use a!defaultValue() for simple cases */
upper(a!defaultValue(local!status, ""))
```

---

## Standard Null Safety Patterns

### Display Patterns (Read-Only)

```sail
/* Display - Integer/Decimal/Text */
if(a!isNotNullOrEmpty(field), field, "N/A")

/* Display - Date */
if(a!isNotNullOrEmpty(field), text(todate(field), "MMM d, yyyy"), "N/A")

/* Display - DateTime */
if(a!isNotNullOrEmpty(field), text(field, "MMM d, yyyy h:mm a"), "N/A")

/* Display - Time */
if(a!isNotNullOrEmpty(field), text(field, "h:mm a"), "N/A")

/* Display - User (firstName + lastName) */
if(
  a!isNotNullOrEmpty(userId),
  trim(user(userId, "firstName") & " " & user(userId, "lastName")),
  "–"
)
```

### Concatenation Patterns

#### Choosing Between & and concat()

**Use `&` operator** (simpler, more readable):
- When joining a few values with literals or spaces
- When readability matters (most common cases)
- Examples: `"Dear " & pv!name & ","` or `field1 & " " & field2`

**Use `concat()` function** (handles arrays):
- When concatenating arrays of text
- When joining many dynamic values without separators
- Example: `concat({"a","b","c"}, {"d","e","f"})` returns `"abcdef"`

Both handle null safely with `a!defaultValue()`:

```sail
/* ✅ Using & operator (simpler for few values) */
a!defaultValue(field1, "") & " " & a!defaultValue(field2, "")

/* ✅ Using concat() function (better for arrays) */
concat(
  a!defaultValue(field1, ""),
  " ",
  a!defaultValue(field2, "")
)

/* Concat with formatting */
"PREFIX-" & if(a!isNotNullOrEmpty(field), field, "N/A")
```

### Array Operations

```sail
/* Protect array references */
length(a!defaultValue(local!items, {}))

/* Array property access (with short-circuit pattern) */
if(
  if(
    a!isNotNullOrEmpty(local!items),
    local!items.type = "Contract",
    false
  ),
  /* Then branch */,
  /* Else branch */
)
```

---

## Null Safety for Computed Variables

**Computed variables that derive from empty arrays require special null checking with nested if() statements.**

**⚠️ IMPORTANT:** SAIL's `and()` and `or()` functions **DO NOT short-circuit**. For detailed explanation, see [short-circuit-patterns.md](short-circuit-patterns.md).

### Pattern for Null-Safe Property Access on Computed Variables

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
  /* Then branch */,
  /* Else branch */
)
```

### Common Scenarios Requiring Nested if()

**1. Computed variables from selections:**

```sail
local!selectedItemIds: {},
local!selectedItems: a!forEach(
  items: local!selectedItemIds,
  expression: index(local!allItems, wherecontains(fv!item, local!allItems.id), null)
),

/* ✅ Accessing properties */
if(
  if(
    a!isNotNullOrEmpty(local!selectedItems),
    length(intersection(local!selectedItems.type, {"Contract"})) > 0,
    false
  ),
  /* Show additional fields */,
  {}
)
```

**2. Filtered or derived arrays:**

```sail
local!activeUsers: a!forEach(
  items: local!allUsers,
  expression: if(fv!item.status = "Active", fv!item, null)
),

/* ✅ Checking properties */
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

---

## Email Validation Pattern

Since SAIL doesn't support regex, use this pattern for robust email validation:

```sail
/* Returns true if valid, false if invalid */
a!localVariables(
  local!trimmed: trim(local!email),
  if(
    or(
      a!isNullOrEmpty(local!trimmed),
      len(local!trimmed) > 255,
      length(split(local!trimmed, " ")) > 1,
      count(split(local!trimmed, "@")) <> 2
    ),
    false(),
    a!localVariables(
      local!localPart: split(local!trimmed, "@")[1],
      local!domainPart: split(local!trimmed, "@")[2],
      if(
        or(
          length(split(local!domainPart, ".")) < 2,
          contains(split(local!localPart, "."), ""),
          contains(split(local!domainPart, "."), ""),
          not(isnull(stripwith(lower(local!domainPart), "abcdefghijklmnopqrstuvwxyz1234567890-."))),
          not(isnull(stripwith(lower(local!localPart), "abcdefghijklmnopqrstuvwxyz1234567890-._+'&%")))
        ),
        false(),
        true()
      )
    )
  )
)
```

**What it validates:**
- Not null, max 255 chars, no spaces, exactly one `@`
- Domain has at least one `.`, no empty segments (catches `..` or leading/trailing dots)
- Valid characters only (alphanumeric + allowed special chars)

**Usage in expression rule:**
```sail
rule!validateEmail(email: local!email)  /* Returns true/false */

/* Use result in conditional logic */
if(
  rule!validateEmail(email: local!inputEmail),
  /* Valid email */,
  /* Invalid email */
)
```

**For other format validations (phone, SSN, zip code, etc.):**
- Use this email pattern as a reference example
- Replace `regexmatch()` thinking with SAIL string functions: `split()`, `stripwith()`, `contains()`, `find()`, `len()`, `count()`
- Break format rules into checkable conditions (length, character sets, required segments)

---

## Fallback Value Rules

| **Context** | **Fallback** | **Reason** |
|------------|-------------|-----------|
| Display text | `"N/A"` or `"–"` | User-facing text |
| Boolean logic (not, showWhen) | `false()` | Safe default for boolean context |
| Array operations | `{}` | Empty array prevents crashes |
| User display | `"–"` (en dash) | Consistent with Appian patterns |

---

## Quick Reference Table

| **Scenario** | **Protected Pattern** | **Fallback** |
|-------------|----------------------|-------------|
| **Display - Integer** | `if(a!isNotNullOrEmpty(field), field, "N/A")` | "N/A" |
| **Display - Decimal** | `if(a!isNotNullOrEmpty(field), field, "N/A")` | "N/A" |
| **Display - Text** | `if(a!isNotNullOrEmpty(field), field, "N/A")` | "N/A" |
| **Display - Date** | `if(a!isNotNullOrEmpty(field), text(todate(field), "MMM d, yyyy"), "N/A")` | "N/A" |
| **Display - DateTime** | `if(a!isNotNullOrEmpty(field), text(field, "MMM d, yyyy h:mm a"), "N/A")` | "N/A" |
| **Display - Time** | `if(a!isNotNullOrEmpty(field), text(field, "h:mm a"), "N/A")` | "N/A" |
| **user() function** | `if(a!isNotNullOrEmpty(userId), trim(user(userId, "firstName") & " " & user(userId, "lastName")), "–")` | "–" |
| **Concat - with field** | `"PREFIX-" & if(a!isNotNullOrEmpty(field), field, "N/A")` | "N/A" |
| **Concat - without formatting** | `a!defaultValue(field1, "") & " " & a!defaultValue(field2, "")` | "" |
| **not() function** | `not(a!defaultValue(var, false()))` | false() |
| **Array length** | `length(a!defaultValue(array, {}))` | {} |
| **Property access** | Use nested if() (see short-circuit-patterns.md) | false |

---

## Summary

**🚨 MANDATORY RULE:**
> **Always check for null BEFORE calling functions that reject null (user, text, not, string manipulation). The a!defaultValue() wrapper alone is insufficient for these functions.**

**Key Patterns:**
1. **user()** - Check null BEFORE calling, use firstName + lastName, fallback to "–"
2. **not()** - Wrap variable in a!defaultValue(var, false())
3. **text()** - Check null BEFORE formatting dates/numbers
4. **String functions** - Check null BEFORE calling upper/lower/left/right/find
5. **Computed variables** - Use nested if() for property access (see short-circuit-patterns.md)
6. **Display fields** - Fallback to "N/A" or "–" for user-facing text
