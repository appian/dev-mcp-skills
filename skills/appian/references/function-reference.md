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

## Complete Function Reference

**Quick lookup table with official signatures from Appian documentation.**

For detailed usage patterns, examples, and gotchas, see the sections below.

**Documentation URLs:** Full documentation available at `https://docs.appian.com/suite/help/{VERSION}/{filename}`  
Where **{VERSION}** is the Appian version configured in [SKILL.md Configuration section](../SKILL.md#configuration) (currently 26.5), and **{filename}** is from the Documentation column below.

| Function | Signature | Returns | Documentation | Purpose |
|----------|-----------|---------|---------------|---------|
| a!addDateTime() | a!addDateTime(startDateTime, years, months, days, hours, minutes, seconds, useProcessCalendar, processCalendarName) | Date and Time | fnc_date_and_time_adddatetime.html | Adds the specified increments of time to the startDateTime and returns a date and time value |
| a!aggregationFields() | a!aggregationFields(groupings, measures) | Aggregation Fields | fnc_scripting_a_aggregationfields.html | Defines query aggregations against record data using groupings and measures within a!queryRecordType() |
| a!defaultValue() | a!defaultValue(value, default) | Any Type | fnc_informational_a_defaultvalue.html | Returns a default value when the specified value is null or empty |
| a!flatten() | a!flatten(array) | Any Type Array | fnc_array_a_flatten.html | Converts an array that contains other arrays into an array of single items |
| a!forEach() | a!forEach(items, expression) | Any Type Array | fnc_looping_a_foreach.html | Evaluates an expression for each item in a list and returns a new array of the results |
| a!fromJson() | a!fromJson(jsonText) | Dictionary/List | fnc_system_a_fromjson.html | Converts a JSON string into an Appian value with automatic casting to appropriate data types |
| a!grouping() | a!grouping(field, interval, alias, formatValue) | Grouping | Grouping_Component.html | Determines the fields to group by in a query or chart that uses a record type as the source |
| a!isNotNullOrEmpty() | a!isNotNullOrEmpty(value) | Boolean | fnc_informational_isnotnullorempty.html | Returns false if the value is null, an empty string, or an empty list; otherwise returns true |
| a!isNullOrEmpty() | a!isNullOrEmpty(value) | Boolean | fnc_informational_isnullorempty.html | Returns true if the value is null, an empty string, or an empty list; otherwise returns false |
| a!jsonPath() | a!jsonPath(value, expression) | Text | fnc_system_a_jsonpath.html | Finds and extracts information from a JSON string using JSONPath syntax to navigate through elements and attributes |
| a!keys() | a!keys(value) | List | fnc_informational_a_keys.html | Returns the keys of the provided map, dictionary, CDT, or record |
| a!localVariables() | a!localVariables(localVar1, localVarN, expression) | Any Type | fnc_evaluation_a_localvariables.html | Defines one or more local variables for use within an expression |
| a!map() | a!map(key1, keyN) | Map | fnc_system_map.html | Creates a map of values with each value stored at the corresponding string key |
| a!match() | a!match(value, equals, then, whenTrue, then, default) | Any Type | fnc_logical_match.html | Evaluates a value against multiple conditions and returns a result based on the first match found |
| a!measure() | a!measure(field, function, alias, label, filters, formatValue) | Measure | Measure_Component.html | Determines the numerical values to display on a query, chart, or KPI by performing calculations on record fields |
| a!pagingInfo() | a!pagingInfo(startIndex, batchSize, sort) | PagingInfo | fnc_system_a_paginginfo.html | Creates a PagingInfo value for controlling data pagination and sorting in grids, queries, and data subsets |
| a!queryFilter() | a!queryFilter(field, operator, value, applyWhen) | QueryFilter | fnc_system_a_queryfilter.html | Creates a QueryFilter value for filtering record data or data store entities in queries, grids, charts, and user filters |
| a!queryLogicalExpression() | a!queryLogicalExpression(operator, logicalExpressions, filters, ignoreFiltersWithEmptyValues) | LogicalExpression | fnc_system_a_querylogicalexpression.html | Creates a LogicalExpression that determines how multiple filter conditions are combined in a query using AND, OR, or AND_ALL operators |
| a!queryRecordByIdentifier() | a!queryRecordByIdentifier(recordType, identifier, fields, relatedRecordData) | Record Type | fnc_system_a_queryrecordbyidentifier.html | Executes a query on a specific record identifier and returns that record's data, including optional related record data |
| a!queryRecordType() | a!queryRecordType(recordType, fields, filters, pagingInfo, fetchTotalCount, relatedRecordData) | Map with fields: success (Boolean), data (List of Records or List of Maps), startIndex (Integer), batchSize (Integer), sort (List of SortInfo), totalCount (Integer), identifiers (List of Integer or List of Text), errorCode (Text) | fnc_system_queryrecordtype.html | Executes a query on a given record type and returns the result, supporting both selection and aggregation of record data |
| a!recordData() | a!recordData(recordType, filters, relatedRecordData, fields) | RecordData | fnc_system_recorddata.html | References a set of records from a record type and allows additional filtering in read-only grids, charts, or selection components |
| a!relatedRecordData() | a!relatedRecordData(relationship, limit, sort, filters) | List of RelatedRecordData | fnc_system_relatedrecorddata.html | References a one-to-many relationship on a record type and enables additional filtering, sorting, and limiting of the related record set |
| a!save() | a!save(target, value) | Save | fnc_evaluation_save.html | Updates a target variable with a given value within a component's saveInto parameter during user interface interactions |
| a!subtractDateTime() | a!subtractDateTime(startDateTime, years, months, days, hours, minutes, seconds, useProcessCalendar, processCalendarName) | Date and Time | fnc_date_and_time_subtractdatetime.html | Subtracts specified time increments from a starting date/time value and returns the resulting date and time |
| a!toJson() | a!toJson(value, removeNullOrEmptyFields) | Text | fnc_system_a_tojson.html | Converts a value into a JSON string, with optional removal of null or empty fields |
| a!update() | a!update(data, index, value) | Any Type | fnc_array_a_update.html | Inserts new values or replaces existing values at the specified index or field name and returns the resulting updated data |
| and() | and(value) | Boolean | fnc_logical_and.html | Returns true if all inputs are true; returns false if at least one input is false |
| append() | append(array, value) | Any Type Array | fnc_array_append.html | Appends a value or values to the given array and returns the resulting array |
| apply() | apply(function, list, context) | Any Type Array | fnc_looping_apply.html | Calls a rule or function for each item in a list and provides any contexts specified |
| choose() | choose(key, choice1, choiceN) | Any Type | fnc_logical_choose.html | Evaluates the choice argument at the given index and returns the result |
| cleanwith() | cleanwith(text, with) | Text | fnc_text_cleanwith.html | Returns the specified text minus any characters not in the list of valid characters |
| concat() | concat(text) | Text | fnc_text_concat.html | Concatenates the specified strings into one string without a separator |
| contains() | contains(array, value) | Boolean | fnc_set_contains.html | Checks whether an array contains the specified value |
| count() | count(value) | Number | fnc_statistical_count.html | Returns the number of items in all arrays passed to the function |
| date() | date(year, month, day) | Date | fnc_date_and_time_date.html | Converts year, month, and day integer values into a date data type |
| dateTime() | dateTime(year, month, day, hour, minute, second, millisecond) | Date and Time | fnc_date_and_time_datetime.html | Converts the given Date and Time into a serial number that holds the Date and Time data type |
| datevalue() | datevalue(value) | Date with Timezone | fnc_date_and_time_datevalue.html | Converts a value to a date |
| difference() | difference(array1, array2) | Any Type Array | fnc_set_difference.html | Returns the values in array1 and not in array2 |
| edate() | edate(starting_date, months) | Date | fnc_date_and_time_edate.html | Returns the date that is the specified number of months before or after a given starting date |
| eomonth() | eomonth(starting_date, months) | Date | fnc_date_and_time_eomonth.html | Returns the date for the last day of the month that is the number of months before or after the given starting date |
| false() | false() | Boolean | fnc_logical_false.html | Returns the Boolean value false |
| filter() | filter(predicate, list, context) | Any Type Array | fnc_looping_filter.html | Calls a predicate for each item in a list and returns any items for which the returned value is true |
| find() | find(search_text, within_text, start_num) | Number | fnc_text_find.html | Searches text for a substring and returns the positional index of the first character of the first match |
| group() | group(groupId, property) | Text | fnc_people_group.html | Returns information for a specified group based on the provided group ID and property parameter |
| if() | if(condition, valueIfTrue, valueIfFalse) | Any Type | fnc_logical_if.html | Returns valueIfTrue if the condition evaluates to true; otherwise returns valueIfFalse |
| index() | index(data, index, default) | Any Type | fnc_array_index.html | Returns data[index] if it is valid or else returns the default value |
| insert() | insert(array, value, index) | Any Type Array | fnc_array_insert.html | Inserts a value into the given array at a specified index and returns the resulting array |
| intersection() | intersection(array1, array2) | Any Type Array | fnc_set_intersection.html | Returns only those elements that appear in all of the given arrays |
| isnull() | isnull(value) | Boolean | fnc_informational_isnull.html | Returns true if value is null, false otherwise |
| left() | left(text, num_chars) | Text | fnc_text_left.html | Returns a specified number of characters from the beginning of a text string |
| len() | len(text) | Number | fnc_text_len.html | Returns the length in characters of the text |
| length() | length(array) | Integer | fnc_array_length.html | Returns the number of elements in an array, excluding null values in most cases |
| loggedInUser() | loggedInUser() | User | fnc_people_loggedinuser.html | Returns the current user logged in to the application |
| lower() | lower(text) | Text | fnc_text_lower.html | Converts all characters in the text into lowercase |
| max() | max(number) | Decimal | fnc_statistical_max.html | Returns the maximum of the specified number(s) |
| merge() | merge(list) | Any Type | fnc_looping_merge.html | Takes a variable number of lists and merges them into a single list that is the size of the largest list provided |
| min() | min(number) | Decimal | fnc_statistical_min.html | Returns the minimum value from the specified number(s) or array of numbers |
| not() | not(value) | Boolean | fnc_logical_not.html | Converts true into false, and false into true |
| now() | now() | Date and Time with Timezone | fnc_date_and_time_now.html | Returns the current date and time as a serial number |
| or() | or(value) | Boolean | fnc_logical_or.html | Returns true if any inputs are true; returns false if all inputs are false |
| property() | property(bean, nameOfProperty, valueIfMissing) | Any Type | fnc_scripting_property.html | Extracts a bean's property value using a specified key name, returning a default value if the property is not present |
| rand() | rand(count) | Decimal | fnc_mathematical_rand.html | Returns a random number between 0 and 1 based on an even probability distribution, which is seeded by the transaction time |
| reduce() | reduce(function, initial, list, context) | Any Type | fnc_looping_reduce.html | Calls a rule or function for each item in a list, passing the result of each call to the next one, and returns the value of the last computation |
| remove() | remove(array, index) | Any Type Array | fnc_array_remove.html | Removes the value at a given index from an array and returns the resulting array |
| right() | right(text, num_chars) | Text | fnc_text_right.html | Returns a specified number of characters from the text, starting from the last character |
| split() | split(text, separator) | Text Array | fnc_text_split.html | Splits text into a list of text elements, delimited by the text specified in the separator |
| stripwith() | stripwith(text, with) | Text | fnc_text_stripwith.html | Returns the provided text with any characters from the invalid characters list removed |
| substitute() | substitute(text, find, replace_with) | Text | fnc_text_substitute.html | Substitutes a specific part of a string with another string |
| text() | text(value, format) | Text | fnc_text_text.html | Converts Number, Date, Time, or Date and time values into formatted text strings |
| time() | time(hour, minute, second, millisecond) | Time | fnc_date_and_time_time.html | Converts the given time into an equivalent time value |
| todate() | todate(value) | Date with Timezone | fnc_conversion_todate.html | Converts a value to Date with Timezone |
| todatetime() | todatetime(value) | Date and Time with Timezone | fnc_conversion_todatetime.html | Converts a value to Date and Time with Timezone format |
| today() | today() | Date with Timezone | fnc_date_and_time_today.html | Returns the current day in Greenwich Mean Time (GMT) |
| tostring() | tostring(value) | Text | fnc_conversion_tostring.html | Converts a value to Text with array values concatenated into one string |
| touniformstring() | touniformstring(value) | Text | fnc_conversion_touniformstring.html | Converts a value or list to text, preserving the original scalar or array structure |
| trim() | trim(text) | Text | fnc_text_trim.html | Removes all unnecessary spaces from the text leaving only single spaces between words |
| true() | true() | Boolean | fnc_logical_true.html | Returns the Boolean value true |
| union() | union(array1, array2) | Any Type Array | fnc_set_union.html | Returns all unique elements from the given arrays |
| upper() | upper(text) | Text | fnc_text_upper.html | Converts all letters in the text into uppercase |
| user() | user(username, property) | Text | fnc_people_user.html | Returns information for a user based on their username and a specified property field |
| where() | where(booleanArray, default) | Integer Array | fnc_array_where.html | Returns the indexes where the values in the input array are true |
| wherecontains() | wherecontains(values, array) | Integer Array | fnc_array_wherecontains.html | Receives one or more values and returns an array of indexes that indicate the position of the values within the array |

---

## When to Search Full Documentation

The table above covers **commonly used functions** loaded for every expression rule. For functions not in this table or when you need deeper details, search the full documentation.

**When to use the Documentation column:**

1. **Complex functions (5+ parameters)** - use Documentation column filename with WebFetch
2. **Validation errors** - function signature correct but behavior unclear
3. **Return type details** - need to understand complex return structures (e.g., a!queryRecordType().data vs .totalCount)
4. **Edge cases** - parameter constraints, null handling, type coercion rules

**How to search:**

Use the **Documentation column filename** to construct the full URL:
```
https://docs.appian.com/suite/help/{VERSION}/{filename}
```

Where:
- **{VERSION}** = The Appian version configured in SKILL.md (currently 26.5)
- **{filename}** = Value from Documentation column (e.g., `fnc_date_and_time_adddatetime.html`)

**Example:**
- Function: `a!addDateTime()`
- Documentation column: `fnc_date_and_time_adddatetime.html`
- Full URL: `https://docs.appian.com/suite/help/26.5/fnc_date_and_time_adddatetime.html`
- Use **WebFetch** tool to retrieve the official documentation page

**For functions not in this table:**
See [expressions.md](expressions.md#when-you-need-more) for the complete documentation lookup workflow.

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
