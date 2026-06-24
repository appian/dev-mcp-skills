# Appian Function Reference

This reference lists Appian functions available for use in expression rules and expressions. Use this as your primary source for function signatures, anti-hallucination checks, and working code examples.

**Related references:**
- For architectural patterns and expression rule management, see [expressions.md](expressions.md)
- For MCP tool usage, see [tools-mcp.md](tools-mcp.md)

---

## Critical Validation Rule

⚠️ **CRITICAL: Check the anti-hallucination list below BEFORE using any function**

### Functions That DO NOT Exist:
- `a!isPageLoad()` - No direct equivalent; SAIL validations evaluate automatically when fields have values
- `property()` - Use dot notation instead: `object.field`
- `a!dateTimeValue()` - Use `dateTime()` instead
- `apply()` - Use `a!forEach()` instead for all iteration
- `regexmatch()`, `regex()`, or any regex functions - SAIL has no regex support; use `split()`, `stripwith()`, `contains()`, `find()` for pattern validation (see [Email Validation Pattern](#email-validation-pattern))

### Functions to Use with Caution:
- `rand()` - Non-deterministic (generates new values on every re-evaluation). Avoid in expression rules unless randomness is explicitly required. Pass seed values via rule inputs if deterministic behavior needed.
- `now()`, `today()` - Non-deterministic (returns current date/time). Safe for audit fields and real-time displays, but avoid in test data or when deterministic results needed.
- `loggedInUser()` - Context-dependent (returns current user). Safe for audit fields and security expressions, but avoid in test scenarios.

### Deprecated/Invalid Parameter Values:
- `batchSize: -1` - Use `batchSize: 5000` (queries), `batchSize: 1` (single aggregations)

---

## Preferred Functions

| Instead Of | Use | Reason |
|------------|-----|--------|
| `isnull()` | `a!isNullOrEmpty()`, `a!isNotNullOrEmpty()` | More comprehensive null checking |
| Infix `&&`, `||` | `and()`, `or()`, `not()` | Appian syntax |
| `map()` | `a!forEach()` | Standard looping |
| `choose()` | `a!match()` | Better pattern matching |

### Other Preferred Patterns:
- **Array Operations**: `append()`, `a!update()` for immutable operations
- **Audit Functions**: `loggedInUser()`, `now()` for audit fields
- **Validator Workarounds**: `union(ri!listInput, ri!listInput)` to force array type in expression rule validators (prevents "Cannot apply operator [IN] to field when comparing to value [0]" errors)

---

## Quick Function Reference by Category

| Category | Functions |
|----------|-----------|
| **Array** | `a!flatten()`, `append()`, `index()`, `length()`, `where()`, `wherecontains()`, `intersection()`, `union()`, `difference()` |
| **Logical** | `and()`, `or()`, `not()`, `if()`, `a!match()` |
| **Null Checking** | `a!isNullOrEmpty()`, `a!isNotNullOrEmpty()`, `a!defaultValue()` |
| **Looping** | `a!forEach()`, `filter()`, `reduce()`, `merge()` |
| **Text** | `concat()`, `find()`, `left()`, `right()`, `len()`, `substitute()`, `upper()`, `lower()`, `trim()`, `split()`, `stripwith()`, `count()` |
| **Date/Time** | `today()`, `now()`, `dateTime()`, `date()`, `time()`, `todate()`, `todatetime()`, `a!addDateTime()`, `a!subtractDateTime()` |
| **JSON** | `a!toJson()`, `a!fromJson()`, `a!jsonPath()` |
| **User/System** | `loggedInUser()`, `user()`, `group()` |
| **Query** | `a!queryRecordType()`, `a!recordData()`, `a!queryFilter()`, `a!pagingInfo()`, `a!aggregationFields()`, `a!measure()`, `a!grouping()` |

---

## JSON Functions

```sail
/* Convert to JSON */
a!toJson(
  value: a!map(name: "John", age: 30),
  removeNullOrEmptyFields: true
)

/* Parse from JSON */
a!fromJson('{"name":"John","age":30}')

/* Extract with JSONPath */
a!jsonPath(json: local!data, expression: "$.employees[0].name")
```

---

## contains() Usage

```sail
/* Simple array check */
contains({"id", "title"}, "title")  /* Returns true */

/* Check in map arrays (mock data) */
contains(
  local!items.name,
  "Alice"
)

/* Check in record arrays */
contains(
  local!records['recordType!Employee.fields.firstName'],
  "Alice"
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

**Usage in validations parameter:**
```sail
validations: if(
  /* email validation pattern returns false */,
  "Please enter a valid email address",
  {}
)
```

**For other format validations (phone, SSN, zip code, etc.):**
- Use this email pattern as a reference example
- Replace `regexmatch()` thinking with SAIL string functions: `split()`, `stripwith()`, `contains()`, `find()`, `len()`, `count()`
- Break format rules into checkable conditions (length, character sets, required segments)

---

## Appian-Specific Gotchas

### find() - 1-indexed, Returns 0 (not -1)

```sail
/* ✅ CORRECT - Check for > 0 */
if(find("@", ri!email) > 0, "has @", "missing @")

/* ❌ WRONG - JavaScript habit (checking for -1) */
if(find("@", ri!email) <> -1, "has @", "missing @")  /* WRONG! Returns 0, not -1 */

/* Returns position (1-indexed) or 0 if not found */
find("@", "user@example.com")  /* Returns 5 */
find("z", "hello")  /* Returns 0 (NOT -1!) */
```

### union() - Validator Array Coercion Workaround

When passing list-type rule inputs to validators in expression rules, Appian's validator sometimes treats them as scalar 0, causing errors like "Cannot apply operator [IN] to field [id] when comparing to value [0]".

**Workaround:** Use `union(ri!listInput, ri!listInput)` to force array type:

```sail
a!queryRecordType(
  recordType: recordType!Case,
  filters: a!queryFilter(
    field: recordType!Case.fields.id,
    operator: "in",
    value: union(ri!caseIds, ri!caseIds)  /* Forces array type for validator */
  )
)
```

This pattern was discovered in Test 6 of Phase 3 testing and is specific to expression rule validators.

### Short-Circuit Evaluation - Use if(), not and()

SAIL's `and()` and `or()` functions do NOT short-circuit - they evaluate ALL arguments even if the result is already determined. This causes crashes when checking null values:

```sail
/* ❌ WRONG - and() evaluates both conditions (crashes if null) */
and(a!isNotNullOrEmpty(local!data), local!data.type = "Contract")  /* CRASHES! */

/* ✅ CORRECT - if() short-circuits (safe) */
if(a!isNotNullOrEmpty(local!data), local!data.type = "Contract", false())
```

For complete null safety patterns, see [expressions.md](expressions.md).

---

## Array Functions Detail

### index()

```sail
/* Get single item by position */
index(local!items, 1, null)  /* First item, null if not found */

/* Get property from array */
index(local!items, "name", {})  /* All name values, empty array if not found */

/* Safe access with wherecontains */
index(local!items, wherecontains(local!targetId, local!items.id), null)
```

### wherecontains()

```sail
/* Find indices where value appears */
wherecontains(local!searchValue, local!array)  /* Returns array of indices */

/* Find matching items */
index(
  local!items,
  wherecontains(local!targetId, local!items.id),
  null
)
```

### a!forEach()

```sail
a!forEach(
  items: local!data,
  expression: a!map(
    id: fv!item.id,
    label: fv!item.name,
    isFirst: fv!isFirst,
    isLast: fv!isLast,
    position: fv!index,
    total: fv!itemCount
  )
)
```

**Available function variables:**
- `fv!item` - Current item
- `fv!index` - Current position (1-based)
- `fv!isFirst` - True if first item
- `fv!isLast` - True if last item
- `fv!itemCount` - Total number of items

---

## a!match() Pattern Matching

```sail
/* Simple status-based styling */
a!match(
  value: local!status,
  equals: "Active", "#059669",
  equals: "Pending", "#D97706",
  equals: "Inactive", "#6B7280",
  default: "#000000"
)

/* With whenTrue for complex conditions */
a!match(
  value: true(),
  whenTrue: local!score >= 90, "A",
  whenTrue: local!score >= 80, "B",
  whenTrue: local!score >= 70, "C",
  default: "F"
)
```

---

## Type Conversion Functions

| Function | Returns | Use Case |
|----------|---------|----------|
| `tointeger(value)` | Integer | IDs, counts, interval-to-number |
| `todecimal(value)` | Decimal | Currency, percentages |
| `tostring(value)` | Text (single string) | Display text (merges arrays!) |
| `touniformstring(array)` | Text array | Preserve array structure |
| `todate(value)` | Date | Date casting |
| `todatetime(value)` | DateTime | DateTime casting |
| `toboolean(value)` | Boolean | Flag conversion |
| `touser(value)` | User | User reference |
| `togroup(value)` | Group | Group reference |

⚠️ **CRITICAL**: `tostring({1, 2})` returns `"1; 2"` (single string). Use `touniformstring({1, 2})` to get `{"1", "2"}` (array).

---

## Mathematical & Random Functions

### rand()

**Syntax:**
- `rand()` - Returns a single decimal between 0 and 1 (e.g., 0.3483318)
- `rand(n)` - Returns an array of n decimals between 0 and 1

**Examples:**
```sail
rand()  /* Returns: 0.3483318 */
rand(5)  /* Returns: {0.1814373, 0.8513633, 0.9319652, 0.1100233, 0.5996339} */
tointeger(rand() * 100) + 1  /* Random integer 1-100 */
```

**⚠️ Use with caution in expression rules** - generates new values on every re-evaluation. Only use when randomness is explicitly required. For deterministic behavior, pass seed values via rule inputs.

---

## Null Checking Functions

```sail
/* Check if null or empty */
a!isNullOrEmpty(value)  /* Returns true if null, "", or {} */

/* Check if has value */
a!isNotNullOrEmpty(value)  /* Returns true if has content */

/* Provide default for null */
a!defaultValue(value, fallback)  /* Returns fallback if value is null */
```

---

## Query Functions (Record Data)

### a!queryRecordType()

```sail
a!queryRecordType(
  recordType: 'recordType!Employee',
  fields: {
    'recordType!Employee.fields.firstName',
    'recordType!Employee.fields.lastName'
  },
  filters: a!queryLogicalExpression(
    operator: "AND",
    filters: {
      a!queryFilter(
        field: 'recordType!Employee.fields.status',
        operator: "=",
        value: "Active"
      )
    }
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100)
)
```

### a!recordData() (for grids)

```sail
a!gridField(
  data: a!recordData(
    recordType: 'recordType!Employee',
    filters: a!queryFilter(...)
  ),
  columns: {...}
)
```

### a!aggregationFields() (for charts/KPIs)

```sail
a!queryRecordType(
  recordType: 'recordType!Order',
  fields: a!aggregationFields(
    groupings: {
      a!grouping(
        field: 'recordType!Order.fields.status',
        alias: "statusName"
      )
    },
    measures: {
      a!measure(
        function: "COUNT",
        field: 'recordType!Order.fields.id',  /* field is REQUIRED even for COUNT */
        alias: "orderCount"
      ),
      a!measure(
        function: "SUM",
        field: 'recordType!Order.fields.amount',
        alias: "totalAmount"
      )
    }
  )
)
```

**Valid a!measure() functions:**
- `"COUNT"` - Count records
- `"SUM"` - Sum numeric field
- `"MIN"` - Minimum value
- `"MAX"` - Maximum value
- `"AVG"` - Average numeric field
- `"DISTINCT_COUNT"` - Count distinct values
