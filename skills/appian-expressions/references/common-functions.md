# Common Appian Functions

## Logical Functions

| Function | Signature | Description |
|---|---|---|
| `if()` | `if(condition, valueIfTrue, valueIfFalse)` | Returns `valueIfTrue` when condition is true, otherwise `valueIfFalse`. |
| `and()` | `and(value1, value2, ...)` | Returns `true` only if ALL arguments are true. |
| `or()` | `or(value1, value2, ...)` | Returns `true` if ANY argument is true. |
| `not()` | `not(value)` | Inverts a Boolean — `true` becomes `false` and vice versa. |
| `true()` | `true()` | Returns the Boolean value `true`. |
| `false()` | `false()` | Returns the Boolean value `false`. |
| `choose()` | `choose(index, choice1, choice2, ...)` | Evaluates and returns the choice at the given 1-based index. |
| `a!match()` | `a!match(value, equals1, then1, ..., default)` | Pattern matching — returns the `then` value for the first matching `equals`. |

### Examples

```
/* Nested conditions — prefer choose() or a!match() for clarity */
if(ri!status = "Open", "Active", if(ri!status = "Closed", "Archived", "Unknown"))

/* Equivalent using a!match() */
a!match(
  value: ri!status,
  equals: "Open", then: "Active",
  equals: "Closed", then: "Archived",
  default: "Unknown"
)

/* Combined boolean logic */
and(
  a!isUserMemberOfGroup(loggedInUser(), cons!ADMIN_GROUP),
  not(isnull(rv!record[recordType!Case.fields.assignedTo]))
)
```

## Null and Informational Functions

| Function | Signature | Description |
|---|---|---|
| `isnull()` | `isnull(value)` | Returns `true` if value is null. |
| `a!isNullOrEmpty()` | `a!isNullOrEmpty(value)` | Returns `true` if value is null, empty string, or empty list. |
| `a!isNotNullOrEmpty()` | `a!isNotNullOrEmpty(value)` | Returns `true` if value is not null, not empty string, and not empty list. |
| `a!defaultValue()` | `a!defaultValue(value, default)` | Returns `value` if not null/empty, otherwise returns `default`. |
| `null()` | `null()` | Returns a null value. |
| `typeof()` | `typeof(value)` | Returns the type number of a value. |
| `typename()` | `typename(typeNumber)` | Returns the type name for a given type number. |

### Examples

```
/* Guard against null */
a!defaultValue(ri!searchText, "")

/* Null-safe display */
if(a!isNullOrEmpty(ri!middleName), ri!firstName & " " & ri!lastName, ri!firstName & " " & ri!middleName & " " & ri!lastName)

/* Prefer a!isNotNullOrEmpty over manual null checks */
a!isNotNullOrEmpty(ri!input)
/* Instead of: not(or(tostring(ri!input) = "", isnull(ri!input))) */
```

## Text Functions

| Function | Signature | Description |
|---|---|---|
| `concat()` | `concat(value1, value2, ...)` | Concatenates values into a single text string. |
| `char()` | `char(code)` | Returns the character for a Unicode code point. `char(10)` = newline. |
| `len()` | `len(text)` | Returns the number of characters in a text string. |
| `left()` | `left(text, numChars)` | Returns the leftmost characters. |
| `right()` | `right(text, numChars)` | Returns the rightmost characters. |
| `mid()` | `mid(text, startIndex, numChars)` | Returns characters from the middle of a text string. |
| `find()` | `find(searchText, withinText, startIndex)` | Returns the position of searchText within withinText (0 if not found). |
| `search()` | `search(searchText, withinText, startIndex)` | Like `find()` but case-insensitive. |
| `substitute()` | `substitute(text, oldText, newText)` | Replaces occurrences of oldText with newText. |
| `trim()` | `trim(text)` | Removes leading and trailing whitespace. |
| `lower()` | `lower(text)` | Converts text to lowercase. |
| `upper()` | `upper(text)` | Converts text to uppercase. |
| `proper()` | `proper(text)` | Capitalizes the first letter of each word. |
| `exact()` | `exact(text1, text2)` | Case-sensitive text comparison. Returns Boolean. |
| `text()` | `text(value, format)` | Formats a number or date as text using a format string. |
| `tostring()` | `tostring(value)` | Converts any value to its text representation. |
| `cleanhtml()` | `cleanhtml(text)` | Strips HTML tags from text. |

### Examples

```
/* Build a display name */
ri!firstName & " " & ri!lastName

/* Multi-line text using char(10) for newline */
"Line 1" & char(10) & "Line 2"

/* Format a number as currency */
text(ri!amount, "$#,##0.00")

/* Case-insensitive search */
search("hello", ri!inputText, 1) > 0
```

## Date and Time Functions

| Function | Signature | Description |
|---|---|---|
| `today()` | `today()` | Returns the current date (no time component). |
| `now()` | `now()` | Returns the current date and time. |
| `date()` | `date(year, month, day)` | Constructs a Date from components. |
| `datetime()` | `datetime(year, month, day, hour, minute, second)` | Constructs a Date and Time. |
| `time()` | `time(hour, minute, second)` | Constructs a Time value. |
| `year()` | `year(date)` | Extracts the year from a date. |
| `month()` | `month(date)` | Extracts the month (1-12) from a date. |
| `day()` | `day(date)` | Extracts the day of the month from a date. |
| `hour()` | `hour(dateTime)` | Extracts the hour from a date and time. |
| `minute()` | `minute(dateTime)` | Extracts the minute from a date and time. |
| `second()` | `second(dateTime)` | Extracts the second from a date and time. |
| `weekday()` | `weekday(date)` | Returns the day of the week (1=Sunday through 7=Saturday). |
| `edate()` | `edate(startDate, months)` | Adds months to a date. |
| `eomonth()` | `eomonth(startDate, months)` | Returns the last day of the month, offset by months. |
| `networkdays()` | `networkdays(startDate, endDate)` | Returns the number of working days between two dates. |
| `datetext()` | `datetext(date, format)` | Formats a date as text using a format string. |
| `todate()` | `todate(value)` | Converts a value to Date. |
| `todatetime()` | `todatetime(value)` | Converts a value to Date and Time. |

### Examples

```
/* Days until deadline */
ri!deadline - today()

/* Is the date in the past? */
ri!dueDate < today()

/* Format a date for display */
datetext(ri!createdDate, "MM/dd/yyyy")

/* Add 30 days to a date */
ri!startDate + 30

/* Business days between dates */
networkdays(ri!startDate, ri!endDate)
```

## List and Array Functions

| Function | Signature | Description |
|---|---|---|
| `a!forEach()` | `a!forEach(items, expression)` | Evaluates expression for each item. Use `fv!item`, `fv!index`, `fv!isFirst`, `fv!isLast`, `fv!itemCount`. |
| `index()` | `index(list, index, default)` | Returns the item at the given index, or default if out of bounds. |
| `length()` | `length(list)` | Returns the number of items in a list. |
| `count()` | `count(list)` | Returns the number of non-null items in a list. |
| `append()` | `append(list, value)` | Returns a new list with value added at the end. |
| `insert()` | `insert(list, value, index)` | Returns a new list with value inserted at the given index. |
| `remove()` | `remove(list, index)` | Returns a new list with the item at index removed. |
| `where()` | `where(booleanList)` | Returns the indices where the Boolean list is true. |
| `wherecontains()` | `wherecontains(values, list)` | Returns indices in list that match any of the given values. |
| `contains()` | `contains(list, value)` | Returns true if the list contains the value. |
| `distinct()` | `distinct(list)` | Returns a list with duplicates removed. |
| `sort()` | `sort(list, ascending)` | Sorts a list. |
| `union()` | `union(list1, list2)` | Returns the union of two lists (distinct values from both). |
| `intersection()` | `intersection(list1, list2)` | Returns values present in both lists. |
| `difference()` | `difference(list1, list2)` | Returns values in list1 that are not in list2. |
| `joinarray()` | `joinarray(list, separator)` | Joins list items into a single text string with separator. |
| `a!flatten()` | `a!flatten(list)` | Flattens nested lists into a single-level list. |
| `merge()` | `merge(list1, list2, ...)` | Combines lists element-wise into a list of lists. |
| `enumerate()` | `enumerate(count)` | Returns a list of integers from 0 to count-1. |
| `repeat()` | `repeat(count, value)` | Returns a list with value repeated count times. |
| `all()` | `all(predicate, list)` | Returns true if predicate is true for all items. |
| `any()` | `any(predicate, list)` | Returns true if predicate is true for any item. |
| `reduce()` | `reduce(rule, initial, list)` | Reduces a list to a single value by applying rule cumulatively. |

### Examples

```
/* Transform a list of records to a list of names */
a!forEach(
  items: ri!employees,
  expression: fv!item[recordType!Employee.fields.firstName] & " " & fv!item[recordType!Employee.fields.lastName]
)

/* Safe index access with default */
index(ri!myList, 1, "N/A")

/* Filter indices where status is "Open" */
index(
  ri!cases,
  where(
    a!forEach(
      items: ri!cases,
      expression: fv!item[recordType!Case.fields.status] = "Open"
    )
  ),
  {}
)

/* Join list into comma-separated text */
joinarray(ri!tags, ", ")
```

## Record Type and Data Functions

| Function | Signature | Description |
|---|---|---|
| `a!queryRecordType()` | `a!queryRecordType(recordType, fields, filters, pagingInfo)` | Queries records from a record type. Returns a DataSubset. |
| `a!recordData()` | `a!recordData(recordType, filters)` | Creates a record data source for grids and charts. |
| `a!queryFilter()` | `a!queryFilter(field, operator, value)` | Creates a filter for record queries. |
| `a!queryLogicalExpression()` | `a!queryLogicalExpression(operator, filters, logicalExpressions)` | Combines filters with AND/OR logic. |
| `a!aggregationFields()` | `a!aggregationFields(groupings, measures)` | Defines grouping and aggregation for record queries. |
| `a!grouping()` | `a!grouping(field, alias)` | Groups query results by a field. |
| `a!measure()` | `a!measure(field, function, alias)` | Aggregates a field (SUM, COUNT, AVG, MIN, MAX). |
| `a!pagingInfo()` | `a!pagingInfo(startIndex, batchSize, sort)` | Defines pagination for queries. |
| `a!sortInfo()` | `a!sortInfo(field, ascending)` | Defines sort order for queries. |
| `a!map()` | `a!map(key1: value1, key2: value2)` | Creates a map (dictionary) of key-value pairs. |
| `cast()` | `cast(typeNumber, value)` | Casts a value to a different type. |
| `a!isUserMemberOfGroup()` | `a!isUserMemberOfGroup(username, group)` | Returns true if the user is a member of the group. |
| `loggedInUser()` | `loggedInUser()` | Returns the current logged-in user. |

### Examples

```
/* Query all open cases */
a!queryRecordType(
  recordType: recordType!Case,
  filters: a!queryFilter(
    field: recordType!Case.fields.status,
    operator: "=",
    value: "Open"
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 50)
)

/* Combined filter with AND */
a!queryLogicalExpression(
  operator: "AND",
  filters: {
    a!queryFilter(field: recordType!Case.fields.status, operator: "=", value: "Open"),
    a!queryFilter(field: recordType!Case.fields.priority, operator: "=", value: "High")
  }
)

/* Check if current user is an admin */
a!isUserMemberOfGroup(loggedInUser(), cons!ADMIN_GROUP)

/* Aggregate: count cases by status */
a!queryRecordType(
  recordType: recordType!Case,
  fields: a!aggregationFields(
    groupings: a!grouping(field: recordType!Case.fields.status, alias: "status"),
    measures: a!measure(field: recordType!Case.fields.id, function: "COUNT", alias: "count")
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100)
)
```

## Conversion and Casting Functions

| Function | Signature | Description |
|---|---|---|
| `tointeger()` | `tointeger(value)` | Converts to Integer (truncates decimals). |
| `todecimal()` | `todecimal(value)` | Converts to Decimal. |
| `tostring()` | `tostring(value)` | Converts to Text. |
| `toboolean()` | `toboolean(value)` | Converts to Boolean. |
| `todate()` | `todate(value)` | Converts to Date. |
| `todatetime()` | `todatetime(value)` | Converts to Date and Time. |
| `totime()` | `totime(value)` | Converts to Time. |
| `touser()` | `touser(value)` | Converts to User (from username text). |
| `togroup()` | `togroup(value)` | Converts to Group. |
| `cast()` | `cast(typeNumber, value)` | General-purpose cast. Use `typeof()` or `'type!{namespace}typeName'` for typeNumber. |
| `a!listType()` | `a!listType(typeNumber)` | Returns the list type number for casting lists. |

### Examples

```
/* Cast a record to a map */
cast(typeof(a!map()), rv!record)

/* Cast a map to a record type */
cast(recordType!Employee, a!map(firstName: "Jane", lastName: "Doe"))

/* Cast a list of mixed values to integers */
cast(a!listType(typeof(1)), {1, 2.5, "3", true})
/* Returns: {1, 2, 3, 1} */
```

## Math Functions

| Function | Signature | Description |
|---|---|---|
| `sum()` | `sum(values...)` | Returns the sum of all values. |
| `average()` | `average(values...)` | Returns the arithmetic mean. |
| `max()` | `max(values...)` | Returns the maximum value. |
| `min()` | `min(values...)` | Returns the minimum value. |
| `abs()` | `abs(number)` | Returns the absolute value. |
| `round()` | `round(number, decimals)` | Rounds to the specified number of decimal places. |
| `ceiling()` | `ceiling(number)` | Rounds up to the nearest integer. |
| `floor()` | `floor(number)` | Rounds down to the nearest integer. |
| `mod()` | `mod(number, divisor)` | Returns the remainder of division. |
| `power()` | `power(base, exponent)` | Returns base raised to the exponent. |
| `sqrt()` | `sqrt(number)` | Returns the square root. |
