# Appian Function Reference

This reference lists Appian functions available for use in expression rules and expressions. Use this as your primary source for function signatures, anti-hallucination checks, and working code examples.

**Related references:**
- For architectural patterns and expression rule management, see [expressions.md](expressions.md)
- For MCP tool usage, see [tools-mcp.md](tools-mcp.md)

**Related pattern files (loaded automatically):**
- [null-safety-patterns.md](null-safety-patterns.md) - Always loaded for null handling patterns
- [short-circuit-patterns.md](short-circuit-patterns.md) - Always loaded for safe conditional evaluation

**For detailed patterns:**
- Load [function-patterns-index.md](function-patterns-index.md) for navigation to:
  - array-patterns.md - Dot notation, wherecontains(), deriving data from IDs
  - date-time-patterns.md - Date/DateTime type matching, query filter compatibility
  - match-foreach-patterns.md - Pattern matching, loop patterns, fv! variables
  - query-record-type-patterns.md - a!queryRecordType() with relationships, paging, sorting, filtering, aggregations

---

## Critical Validation Rule

⚠️ **CRITICAL: Check these lists BEFORE using any function**

### Functions That DO NOT Exist

These functions do not exist in Appian SAIL. Do not use them:

- `regexmatch()`, `regex()` - SAIL has no regex support; use `split()`, `stripwith()`, `contains()`, `find()` for pattern matching (see null-safety-patterns.md for email validation)
- `a!dateTimeValue()` - Use `dateTime()` instead (see date-time-patterns.md)
- `a!isPageLoad()` - No direct equivalent; SAIL validations evaluate automatically when fields have values
- `map()` - Does not exist. Use `a!map()` for creating dictionaries or `a!forEach()` for array iteration

### Functions That Exist But Should NOT Be Used

These functions exist but have better alternatives:

- `apply()` - Technically exists but DO NOT USE. Always use `a!forEach()` instead (better syntax, null handling, and component support)
- `isnull()` - Use `a!isNullOrEmpty()` or `a!isNotNullOrEmpty()` instead (more comprehensive null checking)
- `choose()` - Use `a!match()` instead (better pattern matching)

### Functions to Use with Caution

These functions exist and work correctly but have caveats:

- `rand()` - Non-deterministic (generates new values on every re-evaluation). Avoid in expression rules unless randomness is explicitly required. Pass seed values via rule inputs if deterministic behavior needed.
- `now()`, `today()` - Non-deterministic (returns current date/time). Safe for audit fields and real-time displays, but avoid in test data or when deterministic results needed.
- `loggedInUser()` - Context-dependent (returns current user). Safe for audit fields and security expressions, but avoid in test scenarios.
- `property()` for general object access - Does not work on maps, CDTs, or records. Use dot notation (`object.field`) or `index()` instead. Note: `property()` exists but only for Java beans and `msg!properties` - not for typical SAIL data structures.

### Deprecated/Invalid Parameter Values

- `batchSize: -1` - Use `batchSize: 5000` (queries), `batchSize: 1` (single aggregations)

---

## Preferred Functions

| Instead Of | Use | Reason |
|------------|-----|--------|
| `isnull()` | `a!isNullOrEmpty()`, `a!isNotNullOrEmpty()` | More comprehensive null checking |
| Infix `&&`, `||` | `and()`, `or()`, `not()` | Appian syntax |
| `apply()` | `a!forEach()` | Standard looping (apply exists but worse syntax/null handling) |
| `choose()` | `a!match()` | Better pattern matching |

### Other Preferred Patterns:
- **Array Operations**: `append()`, `a!update()` for immutable operations
- **Audit Functions**: `loggedInUser()`, `now()` for audit fields
- **Validator Workarounds**: `union(ri!listInput, ri!listInput)` to force array type in expression rule validators (prevents "Cannot apply operator [IN] to field when comparing to value [0]" errors)

---

## Quick Function Reference by Category

| Category | Functions |
|----------|-----------|
| **Variables & State** | `a!localVariables()`, `a!save()` |
| **Map/Dictionary** | `a!map()`, `a!keys()` |
| **Array** | `a!flatten()`, `append()`, `index()`, `length()`, `where()`, `wherecontains()`, `intersection()`, `union()`, `difference()` |
| **Logical** | `and()`, `or()`, `not()`, `if()`, `a!match()` |
| **Boolean** | `true()`, `false()` (literals `true`, `false` preferred) |
| **Null Checking** | `a!isNullOrEmpty()`, `a!isNotNullOrEmpty()`, `a!defaultValue()` |
| **Looping** | `a!forEach()`, `filter()`, `reduce()`, `merge()` |
| **Text** | `concat()`, `find()`, `left()`, `right()`, `len()`, `substitute()`, `upper()`, `lower()`, `trim()`, `split()`, `cleanwith()`, `stripwith()`, `count()` |
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

## Text Character Filtering: cleanwith() vs stripwith()

**CRITICAL:** These functions have opposite behavior - using the wrong one causes validation bugs.

### cleanwith() - KEEPS characters in the set (removes everything else)

**Use for:** Extracting only allowed characters (e.g., digits from phone number)

```sail
/* Extract digits from phone number */
cleanwith("(555) 555-5555", "0123456789") → "5555555555"

/* Extract alphanumeric only */
cleanwith("Hello-World_123!", "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789")
→ "HelloWorld123"

/* Use case: Count digits in phone number */
local!digitsOnly: cleanwith(ri!phoneNumber, "0123456789"),
local!digitCount: len(local!digitsOnly)
```

### stripwith() - REMOVES characters in the set (keeps everything else)

**Use for:** Removing known characters, detecting invalid characters

```sail
/* Remove allowed characters to find invalid ones */
stripwith("(555) 555-5555", "0123456789()-. +") → ""  /* Empty = all valid */
stripwith("(555) 555-ABCD", "0123456789()-. +") → "ABCD"  /* Invalid chars */

/* Remove formatting from text */
stripwith("Hello-World_123", "-_") → "HelloWorld123"

/* Use case: Validate only allowed characters */
local!invalidChars: stripwith(ri!email, "abcdefghijklmnopqrstuvwxyz0123456789@.-_+"),
local!isValid: a!isNullOrEmpty(local!invalidChars)
```

### Common Mistake: Wrong Function Selection

**❌ WRONG - Using stripwith() to extract digits:**
```sail
/* Intending to extract digits from phone number */
local!digitsOnly: stripwith("(555) 555-5555", "0123456789")
/* Result: "() -" - REMOVES digits, keeps formatting! */
```

**✅ CORRECT - Using cleanwith() to extract digits:**
```sail
local!digitsOnly: cleanwith("(555) 555-5555", "0123456789")
/* Result: "5555555555" - KEEPS digits, removes formatting */
```

### Decision Guide

| Goal | Function | Example |
|------|----------|---------|
| **Extract specific characters** (keep only these) | `cleanwith()` | Extract digits: `cleanwith(text, "0123456789")` |
| **Remove specific characters** (remove these) | `stripwith()` | Remove spaces: `stripwith(text, " ")` |
| **Detect invalid characters** (find what shouldn't be there) | `stripwith()` | `stripwith(text, allowedChars)` returns invalid chars |
| **Count specific characters** | `cleanwith()` then `len()` | `len(cleanwith(text, "0123456789"))` |

### Common Patterns

**Phone number validation:**
```sail
/* Extract digits, check count */
local!digitsOnly: cleanwith(ri!phoneNumber, "0123456789"),
local!digitCount: len(local!digitsOnly),
and(
  local!digitCount >= 10,
  local!digitCount <= 15
)
```

**Character whitelist validation:**
```sail
/* Remove allowed chars, check if anything remains */
local!invalidChars: stripwith(
  ri!input,
  "abcdefghijklmnopqrstuvwxyz0123456789-_"
),
a!isNullOrEmpty(local!invalidChars)  /* true = all valid */
```

**Extract alphanumeric only:**
```sail
cleanwith(
  ri!text,
  "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
)
```

### Memory Aid

- **clean**with() = **clean** up text by keeping **with** these characters → KEEPS
- **strip**with() = **strip** away characters matching **with** these → REMOVES

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

For complete patterns and common scenarios, see [short-circuit-patterns.md](short-circuit-patterns.md).

---

## Array Functions Detail

### index()

**Signature:** `index(data, index, default)`

Returns value(s) at specified index/indices from array, dictionary, map, CDT, or record.

**Parameters:**
- `data` - Array, dictionary, map, CDT, or record
- `index` - Integer (for arrays), text (for dictionaries/maps/fields), or array of indices
- `default` - Optional. Value to return if index invalid (must match element type)

**Returns:** Value at index, or default if invalid

**Basic Usage:**
```sail
/* Array indexing (1-based) */
index({10, 20, 30}, 2, null)  /* Returns 20 */

/* Multiple indices */
index({10, 20, 30}, {1, 3}, null)  /* Returns {10, 30} */

/* Get property from array of maps */
index(local!items, "name", {})  /* All name values */

/* Safe access with wherecontains */
index(local!items, wherecontains(local!targetId, local!items.id), null)
```

**Important Gotchas:**

⚠️ **Dictionary behavior:** When data is a dictionary and index not found, the default is IGNORED and null is returned.

⚠️ **Cannot use in saveInto:** You cannot use `index()` to specify the index for a saveInto parameter. Use square brackets instead:
```sail
/* ❌ WRONG */
a!save(index(local!items, 2), newValue)

/* ✅ CORRECT */
a!save(local!items[2], newValue)
```

**Advanced Usage:**
```sail
/* Index into CDT with field name */
index(ri!customer, "firstName", "Unknown")

/* Index into record with field reference */
index(ri!record, recordType!Case.fields.status, null)

/* Dictionary with mixed key types requires string index */
index({1: "a", "2": "b"}, "1", null)  /* String index required */
```

**For detailed array manipulation patterns, see [array-patterns.md](array-patterns.md).**

### where()

**Signature:** `where(booleanArray, default)`

Returns the 1-based indices where values in the array are true.

**Parameters:**
- `booleanArray` - Array of boolean values to test
- `default` - Optional. Value to return if no true values found

**Returns:** Array of integers (indices where values are true)

**Behavior:**
```sail
/* Basic usage */
where({true, false, true})  /* Returns {1, 3} */

/* With default */
where({false, false}, 999)  /* Returns {999} */
where({false, false})  /* Returns {} (empty) */

/* Null handling */
where({true, false, null, true})  /* Returns {1, 4} (null treated as false) */

/* Common pattern with comparisons */
where(mod({13, 24, 35, 46}, 2) = 0)  /* Returns {2, 4} (even numbers) */

/* Combined with index() */
index(
  ri!employees.firstName,
  where(ri!employees.department = "Finance", -1),
  "None"
)  /* Returns Finance employee names, or "None" if none */
```

**Key Points:**
- Returns 1-based indices (first element is 1)
- Null and empty arrays treated as false
- If no true values and no default: returns `{}`
- If no true values and default specified: returns default

**For wherecontains() patterns and deriving data from ID arrays, see [array-patterns.md](array-patterns.md).**

---

## a!match() and a!forEach() Quick Reference

### a!match() - Pattern Matching

```sail
/* Simple status-based styling */
a!match(
  value: local!status,
  equals: "Active", "#059669",
  equals: "Pending", "#D97706",
  equals: "Inactive", "#6B7280",
  default: "#000000"
)

/* With whenTrue for range-based conditions */
a!match(
  value: local!score,
  whenTrue: fv!value >= 90, "A",
  whenTrue: fv!value >= 80, "B",
  whenTrue: fv!value >= 70, "C",
  default: "F"
)
```

**For complete a!match() patterns (equals vs whenTrue, status lookups, range comparisons), see [match-foreach-patterns.md](match-foreach-patterns.md).**

### a!forEach() - Iteration

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

**For complete fv! variable reference and iteration patterns, see [match-foreach-patterns.md](match-foreach-patterns.md).**

### a!map() - Create Dictionary/Map Structures

**Use for:** Creating key-value pair structures (maps/dictionaries)

**Signature:** `a!map(key1: value1, key2: value2, ...)`

**Returns:** Map (dictionary)

**Common use cases:**
- Structured data for interface local variables
- API request/response payloads
- Organizing related data
- Return values from expression rules

**Basic usage:**
```sail
a!map(
  firstName: "John",
  lastName: "Doe",
  age: 30,
  isActive: true
)
```

**Nested maps:**
```sail
a!map(
  user: a!map(
    firstName: "John",
    lastName: "Doe"
  ),
  address: a!map(
    street: "123 Main St",
    city: "Springfield"
  )
)
```

**Accessing map values:**
```sail
a!localVariables(
  local!user: a!map(
    firstName: "John",
    lastName: "Doe"
  ),
  {
    local!user.firstName,  /* Returns "John" */
    local!user["lastName"]  /* Returns "Doe" */
  }
)
```

**Array of maps:**
```sail
{
  a!map(id: 1, name: "Alice"),
  a!map(id: 2, name: "Bob"),
  a!map(id: 3, name: "Charlie")
}
```

**⚠️ NOT for looping:** `a!map()` creates data structures. Use `a!forEach()` to iterate over arrays.

**Related functions:**
- `a!keys()` - Extract keys from a map
- `index()` - Access map values by key
- `a!toJson()` - Convert map to JSON string

---

## Type Conversion Functions

| Function | Returns | Use Case |
|----------|---------|----------|
| `tointeger(value)` | Integer | IDs, counts, interval-to-number, truncates decimals |
| `todecimal(value)` | Decimal | Currency, percentages |
| `tostring(value)` | Text (single string) | Display text (merges arrays!) |
| `touniformstring(array)` | Text array | Preserve array structure |
| `toboolean(value)` | Boolean | Flag conversion |
| `todate(value)` | Date | Date casting |
| `todatetime(value)` | DateTime | DateTime casting |
| `totime(value)` | Time | Time casting |
| `touser(value)` | User | User reference from username text |
| `togroup(value)` | Group | Group reference |
| `cast(typeNumber, value)` | Varies | General-purpose cast using type number or `'type!{namespace}typeName'` |
| `typeof(value)` | Number | Get the type number of a value |
| `typename(typeNumber)` | Text | Get the type name from a type number |

**Key Considerations:**
- Some casts lose information: `tointeger(123.45)` returns `123` (truncates decimals)
- Comparison operators auto-normalize types: Integer vs. Decimal promotes to Decimal; any type vs. Text promotes to Text
- ⚠️ **CRITICAL**: `tostring({1, 2})` returns `"1; 2"` (single string). Use `touniformstring({1, 2})` to get `{"1", "2"}` (array)

**Examples:**
```sail
/* Basic conversions */
tointeger("123")  /* Returns 123 */
todecimal("123.45")  /* Returns 123.45 */
tointeger(123.45)  /* Returns 123 (truncates!) */

/* Array to string - GOTCHA */
tostring({1, 2, 3})  /* Returns "1; 2; 3" (single string) */
touniformstring({1, 2, 3})  /* Returns {"1", "2", "3"} (array) */

/* Type introspection */
typeof(123)  /* Returns type number for Integer */
typename(typeof(123))  /* Returns "Number (Integer)" */

/* General-purpose casting */
cast(typeof(a!map()), ri!record)  /* Cast record to map */
cast(recordType!Employee, ri!employeeMap)  /* Cast map to record */
```

**For array type initialization (empty arrays), see [array-patterns.md](array-patterns.md).**

**For date/time type compatibility in query filters, see [date-time-patterns.md](date-time-patterns.md).**

---

## Boolean Functions and Literals

**Literal form preferred:** Appian supports both boolean literals and boolean functions. Always prefer literals for clarity and brevity.

```sail
/* ✅ PREFERRED - Literals */
local!isValid: false,
local!isActive: true,
if(local!isValid, "Yes", "No")

/* ✅ ALSO WORKS - Functions (unnecessary verbosity) */
local!isValid: false(),
local!isActive: true(),
if(local!isValid, "Yes", "No")
```

**Available functions:**
- `false()` - Returns boolean false (prefer literal `false`)
- `true()` - Returns boolean true (prefer literal `true`)

**When you might see function form:**
- Legacy code written before literals were common practice
- Code generated by automated tools
- Personal style preferences (both are valid)

**Recommendation:** Use literals (`false`, `true`) in new code for consistency and readability.

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

## Local Variables and State Management

### a!localVariables()

**Use for:** Declaring local variables within an expression to store intermediate values

**Signature:** `a!localVariables(local!var1: value1, local!var2: value2, ..., expression)`

**Returns:** Result of the final expression

**Why use local variables:**
- Avoid re-evaluating expensive expressions (queries, calculations)
- Make complex expressions more readable
- Store intermediate results for reuse

**Basic usage:**
```sail
a!localVariables(
  local!firstName: "John",
  local!lastName: "Doe",
  concat(local!firstName, " ", local!lastName)
)
```

**With queries:**
```sail
a!localVariables(
  local!cases: a!queryRecordType(
    recordType: 'recordType!Case',
    pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100)
  ).data,
  local!activeCount: length(where(local!cases.status = "Active")),
  local!closedCount: length(where(local!cases.status = "Closed")),
  a!map(
    active: local!activeCount,
    closed: local!closedCount,
    total: length(local!cases)
  )
)
```

**⚠️ Local variables are evaluated in order:** Later variables can reference earlier ones, but not vice versa.

```sail
a!localVariables(
  local!x: 10,
  local!y: local!x + 5,  /* ✅ Can reference local!x (defined earlier) */
  local!z: local!y * 2   /* ✅ Can reference local!y (defined earlier) */
)
```

**For detailed patterns, see:** [null-safety-patterns.md](null-safety-patterns.md)

---

### a!save()

**Use for:** Updating variable values in interface interactions (button clicks, field changes)

**Signature:** `a!save(target, value)`

**Parameters:**
- `target` - Variable to update (local! or ri!)
- `value` - New value to assign

**Returns:** Null (used for side effect only)

**Common use case - Button click:**
```sail
a!buttonWidget(
  label: "Submit",
  saveInto: a!save(local!isSubmitted, true)
)
```

**Multiple saves:**
```sail
a!buttonWidget(
  label: "Save",
  saveInto: {
    a!save(local!firstName, ri!firstName),
    a!save(local!lastName, ri!lastName),
    a!save(local!isModified, true)
  }
)
```

**Conditional save:**
```sail
a!buttonWidget(
  label: "Submit",
  saveInto: if(
    local!isValid,
    {
      a!save(local!data, ri!input),
      a!save(local!saved, true)
    },
    a!save(local!error, "Invalid input")
  )
)
```

**Array updates:**
```sail
/* Append to array */
a!save(local!items, append(local!items, ri!newItem))

/* Update specific index */
a!save(local!items[2], "New Value")

/* Remove item */
a!save(local!items, remove(local!items, local!indexToRemove))
```

**Related functions:**
- `a!update()` - Immutable array updates
- `append()` - Add to array
- `remove()` - Remove from array

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
