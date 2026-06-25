# Function Patterns Index

This index provides quick navigation to detailed function patterns for expression rules. Use this to locate the right pattern file for your task.

**Core reference files (always loaded):**
- [function-reference.md](function-reference.md) - Function signatures, anti-hallucination checks, quick examples
- [null-safety-patterns.md](null-safety-patterns.md) - Null handling patterns (user, text, not, validation)
- [short-circuit-patterns.md](short-circuit-patterns.md) - Safe conditional evaluation (nested if() patterns)

**Detailed pattern files (load when needed):**
- [array-patterns.md](array-patterns.md) - Array manipulation, dot notation, deriving data
- [date-time-patterns.md](date-time-patterns.md) - Date/DateTime type matching, query filter compatibility
- [match-foreach-patterns.md](match-foreach-patterns.md) - Pattern matching, iteration, fv! variables
- [query-record-type-patterns.md](query-record-type-patterns.md) - a!queryRecordType() with relationships, paging, sorting, filtering, aggregations

---

## Quick Navigation by Task Type

### When Querying Record Types

**Load:** [query-record-type-patterns.md](query-record-type-patterns.md)

- ✅ Required parameters (fields, pagingInfo, fetchTotalCount)
- ✅ Including related fields via relationships (many-to-one)
- ✅ Paging and batch sizes
- ✅ Sorting (sort vs sorts, inside pagingInfo)
- ✅ Filtering with variables (applyWhen patterns)
- ✅ Nested logical expressions (AND + OR)
- ✅ Aggregation queries for KPIs (COUNT, SUM, AVG with groupings)
- ✅ Result handling (record instances vs maps)
- ✅ Property access patterns by query type
- ✅ Common anti-patterns and fixes

**Example tasks:**
- "How do I query a record type with related fields?"
- "How do I add sorting to a query?"
- "How do I build a KPI count query?"
- "Why are my query results returning null fields?"
- "How do I filter by a variable that might be null?"

---

### When Working with Arrays

**Load:** [array-patterns.md](array-patterns.md)

- ✅ Accessing properties across arrays (dot notation)
- ✅ Deriving full data from ID arrays (a!forEach + index + wherecontains)
- ✅ Finding a single matching item by ID
- ✅ Using wherecontains() correctly (2 parameters, returns indices)
- ✅ Array type initialization (tointeger, touniformstring, etc.)
- ✅ tostring() vs touniformstring() (critical distinction)
- ✅ Array modification functions (append, insert, remove, a!update)
- ✅ Common error scenarios and solutions

**Example tasks:**
- "How do I get all names from an array of maps?"
- "How do I find matching items from a list of IDs?"
- "Why am I getting 'List of Variant' errors?"
- "Should I use tostring() or touniformstring()?"

---

### When Working with Dates and Times

**Load:** [date-time-patterns.md](date-time-patterns.md)

- ✅ Correct date/time functions (dateTime() NOT a!dateTimeValue())
- ✅ Date vs DateTime type distinction (not interchangeable)
- ✅ Date arithmetic (+ and -) vs DateTime arithmetic (a!addDateTime/a!subtractDateTime)
- ✅ Interval return type (date subtraction returns Interval, not Number)
- ✅ Type compatibility for query filters
- ✅ Type conversion between Date and DateTime
- ✅ Function return types (Date vs DateTime)
- ✅ text() function with date/time values (null safety)
- ✅ min()/max() return type casting

**Example tasks:**
- "How do I add 30 days to a date?"
- "Why is my query filter not working with dates?"
- "How do I convert Date to DateTime?"
- "Why am I getting Interval type errors?"

---

### When Working with Pattern Matching and Loops

**Load:** [match-foreach-patterns.md](match-foreach-patterns.md)

- ✅ a!match() for status/category mapping (equals vs whenTrue)
- ✅ Range-based comparisons with whenTrue
- ✅ When to use a!match() vs nested if()
- ✅ a!forEach() iteration patterns
- ✅ Complete fv! variable reference (item, index, isFirst, isLast, itemCount)
- ✅ Common mistakes with fv! variables
- ✅ Short-circuit behavior of a!match()

**Example tasks:**
- "How do I map status codes to colors?"
- "How do I handle numeric ranges/thresholds?"
- "What are the fv! variables in a!forEach()?"
- "How do I add remove buttons to dynamic lists?"
- "Should I use a!match() or nested if()?"

---

### When Handling Null Values

**Load:** [null-safety-patterns.md](null-safety-patterns.md)

- ✅ Functions that reject null (user, text, not, string manipulation)
- ✅ user() function patterns (MUST check null before calling)
- ✅ not() function patterns (wrap in a!defaultValue)
- ✅ text() function with date/time values
- ✅ Null safety for computed variables (nested if() required)
- ✅ Email validation pattern (no regex support)
- ✅ Standard null safety patterns by scenario
- ✅ Fallback value rules

**Example tasks:**
- "How do I display user names safely?"
- "Why is not() failing with null values?"
- "How do I format dates that might be null?"
- "How do I validate email addresses without regex?"

---

### When Using Conditional Logic

**Load:** [short-circuit-patterns.md](short-circuit-patterns.md)

- ✅ The problem (and/or don't short-circuit)
- ✅ The solution (nested if())
- ✅ When to use nested if() vs and()
- ✅ Common scenarios requiring nested if()
- ✅ Pattern for null-safe property access
- ✅ Common mistakes and fixes

**Example tasks:**
- "Why is my property access crashing on empty arrays?"
- "How do I safely check properties on computed variables?"
- "When should I use and() vs nested if()?"
- "Why does and() evaluate all arguments?"

---

## Navigation by Error Type

| Error Message | Load This File |
|---------------|----------------|
| "Invalid index: Cannot index property 'X' of Null" | [short-circuit-patterns.md](short-circuit-patterns.md) |
| "List of Variant" errors | [array-patterns.md](array-patterns.md) |
| "Type mismatch" with Date/DateTime in query filters | [date-time-patterns.md](date-time-patterns.md) |
| "Cannot apply operator" with Interval type | [date-time-patterns.md](date-time-patterns.md) |
| User/text/not function errors with null | [null-safety-patterns.md](null-safety-patterns.md) |
| "tostring() returned single string instead of array" | [array-patterns.md](array-patterns.md) |
| "contains() type mismatch" | [array-patterns.md](array-patterns.md) |
| "wherecontains() invalid parameter count" | [array-patterns.md](array-patterns.md) |
| Property access on empty computed variables | [short-circuit-patterns.md](short-circuit-patterns.md) |

---

## Navigation by Function

| Function | Primary File | Also See |
|----------|-------------|----------|
| `a!queryRecordType()` | [query-record-type-patterns.md](query-record-type-patterns.md) | - |
| `a!recordData()` | [query-record-type-patterns.md](query-record-type-patterns.md) | - |
| `a!match()` | [match-foreach-patterns.md](match-foreach-patterns.md) | [short-circuit-patterns.md](short-circuit-patterns.md) |
| `a!forEach()` | [match-foreach-patterns.md](match-foreach-patterns.md) | [array-patterns.md](array-patterns.md) |
| `user()` | [null-safety-patterns.md](null-safety-patterns.md) | - |
| `not()` | [null-safety-patterns.md](null-safety-patterns.md) | - |
| `text()` | [null-safety-patterns.md](null-safety-patterns.md) | [date-time-patterns.md](date-time-patterns.md) |
| `wherecontains()` | [array-patterns.md](array-patterns.md) | - |
| `index()` | [function-reference.md](function-reference.md) | [array-patterns.md](array-patterns.md) |
| `today()` | [date-time-patterns.md](date-time-patterns.md) | - |
| `now()` | [date-time-patterns.md](date-time-patterns.md) | - |
| `dateTime()` | [date-time-patterns.md](date-time-patterns.md) | - |
| `a!addDateTime()` | [date-time-patterns.md](date-time-patterns.md) | - |
| `a!subtractDateTime()` | [date-time-patterns.md](date-time-patterns.md) | - |
| `tointeger()` | [array-patterns.md](array-patterns.md) | [date-time-patterns.md](date-time-patterns.md) |
| `touniformstring()` | [array-patterns.md](array-patterns.md) | - |
| `tostring()` | [array-patterns.md](array-patterns.md) | - |
| `todate()` | [date-time-patterns.md](date-time-patterns.md) | - |
| `todatetime()` | [date-time-patterns.md](date-time-patterns.md) | - |
| `and()` / `or()` | [short-circuit-patterns.md](short-circuit-patterns.md) | - |
| `if()` | [short-circuit-patterns.md](short-circuit-patterns.md) | [null-safety-patterns.md](null-safety-patterns.md) |

---

## Pattern File Descriptions

### function-reference.md
**Purpose:** Primary function catalog with signatures and quick examples
**Load when:** Need to verify function existence, check parameters, see quick examples
**Line count:** ~500 lines
**Key content:**
- Anti-hallucination list (functions that DON'T exist)
- Preferred functions table
- Quick function reference by category
- Function signatures and basic examples
- Links to detailed pattern files

---

### null-safety-patterns.md
**Purpose:** Comprehensive null handling patterns
**Load when:** Working with functions that reject null (user, text, not)
**Always loaded:** Yes (core safety patterns)
**Line count:** ~300 lines
**Key content:**
- Functions that reject null (user, text, not, string manipulation)
- Critical patterns by function (user, not, text, string functions)
- Null safety for computed variables (nested if() patterns)
- Email validation pattern (no regex support)
- Standard patterns and quick reference table

---

### short-circuit-patterns.md
**Purpose:** Safe conditional evaluation patterns
**Load when:** Need to access properties on potentially empty variables
**Always loaded:** Yes (critical safety pattern)
**Line count:** ~100 lines
**Key content:**
- The problem (and/or don't short-circuit)
- The solution (nested if())
- When to use nested if() vs and()
- Common scenarios (grid selections, array property access, filtered arrays)
- Pattern for null-safe property access

---

### array-patterns.md
**Purpose:** Array operations and data derivation
**Load when:** Working with arrays, property access, ID lookups, type initialization
**Line count:** ~300 lines
**Key content:**
- Dot notation for property access (property() does NOT exist)
- Deriving full data from ID arrays (a!forEach + index + wherecontains)
- Finding single matching item by ID
- wherecontains() correct usage (2 params, returns indices)
- Array type initialization (tointeger, touniformstring, etc.)
- tostring() vs touniformstring() (CRITICAL distinction)
- Common error scenarios and solutions

---

### date-time-patterns.md
**Purpose:** Date/time type handling and compatibility
**Load when:** Working with dates, times, date arithmetic, query filter dates
**Line count:** ~200 lines
**Key content:**
- Correct date/time functions (dateTime() NOT a!dateTimeValue())
- Date vs DateTime type distinction (not interchangeable)
- Date arithmetic (+ and -) vs DateTime arithmetic (a!addDateTime/a!subtractDateTime)
- Interval return type (date subtraction returns Interval)
- Type compatibility for query filters
- Function return types reference
- text() function with date/time values

---

### match-foreach-patterns.md
**Purpose:** Pattern matching and iteration patterns
**Load when:** Need status mapping, range comparisons, loops, fv! variables
**Line count:** ~200 lines
**Key content:**
- a!match() equals vs whenTrue (SCALAR only, not arrays)
- Range-based comparisons with whenTrue
- When to use a!match() vs nested if()
- fv! variables complete reference (item, index, isFirst, isLast, itemCount)
- Common mistakes (0-based vs 1-based, using fv! outside forEach)
- Short-circuit behavior

---

### query-record-type-patterns.md
**Purpose:** Comprehensive a!queryRecordType() patterns for querying record data
**Load when:** Building queries with filters, sorting, relationships, or aggregations
**Line count:** ~650 lines
**Key content:**
- Required parameters (fields, pagingInfo, fetchTotalCount - all mandatory)
- Relationship traversal (accessing many-to-one related fields)
- Paging and sorting (sort goes inside pagingInfo, not separate parameter)
- Advanced filtering (applyWhen for variables, nested logical expressions)
- Aggregation queries for KPIs (COUNT, SUM, AVG with groupings)
- Result handling (record instances vs maps, property access patterns)
- Common anti-patterns (missing fetchTotalCount, wrong parameter names)
- Batch size recommendations by use case

---

### function-patterns-index.md
**Purpose:** Navigation index for detailed patterns
**Load when:** Need to find the right pattern file for your task
**Line count:** ~200 lines (this file)
**Key content:**
- Quick navigation by task type
- Navigation by error type
- Navigation by function
- Pattern file descriptions with when to load

---

## How to Use This Index

### Step 1: Identify Your Task
- Working with arrays? → [array-patterns.md](array-patterns.md)
- Working with dates/times? → [date-time-patterns.md](date-time-patterns.md)
- Need status mapping or loops? → [match-foreach-patterns.md](match-foreach-patterns.md)
- Handling null values? → [null-safety-patterns.md](null-safety-patterns.md)
- Property access on computed variables? → [short-circuit-patterns.md](short-circuit-patterns.md)

### Step 2: Load the Relevant File
- Read the file directly for complete patterns
- Use the "Key content" bullets above to verify it covers your need

### Step 3: Reference Multiple Files if Needed
- Many patterns span multiple files (e.g., array property access needs both array-patterns.md and short-circuit-patterns.md)
- Follow the "Also See" links in the navigation table

---

## Loading Strategy

**Core files (always load):**
1. [function-reference.md](function-reference.md) - Function catalog and quick examples
2. [null-safety-patterns.md](null-safety-patterns.md) - Critical null handling
3. [short-circuit-patterns.md](short-circuit-patterns.md) - Safe conditional evaluation

**Detailed files (load when needed):**
- [query-record-type-patterns.md](query-record-type-patterns.md) - For querying record types
- [array-patterns.md](array-patterns.md) - For array operations
- [date-time-patterns.md](date-time-patterns.md) - For date/time work
- [match-foreach-patterns.md](match-foreach-patterns.md) - For pattern matching and loops

**This index (load when navigating):**
- [function-patterns-index.md](function-patterns-index.md) - To find the right pattern file

---

## Summary

This index helps you quickly locate the detailed patterns you need for expression rule development. The modular structure ensures you can:

1. **Load core safety patterns always** (null-safety, short-circuit)
2. **Load detailed patterns on demand** (arrays, dates, matching)
3. **Navigate by task, error, or function** (use the tables above)
4. **Understand what each file covers** (see descriptions)

**Total line count across all files:** ~1,800 lines (compared to ~500 lines in original function-reference.md)

**Coverage improvement:** ~1,000 lines of patterns added from composer logic-guidelines

**Key improvement:** Modular structure allows targeted loading based on specific needs, reducing cognitive load while maintaining comprehensive coverage.
