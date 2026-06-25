# Date/Time Handling Patterns for Expression Rules

This file documents date and time type handling patterns for expression rules, focusing on type compatibility and correct function usage.

**Related references:**
- [function-reference.md](function-reference.md) - Function signatures and quick examples
- [null-safety-patterns.md](null-safety-patterns.md) - Null handling for date/time values
- [array-patterns.md](array-patterns.md) - Array manipulation patterns

---

## Correct Date/Time Functions

🚨 **CRITICAL**: Use the correct function for date/time creation

```sail
/* ✅ CORRECT - Use dateTime() for specific date/time creation */
dateTime(year(today()), month(today()), 1, 0, 0, 0)  /* Month to Date */

/* ❌ WRONG - a!dateTimeValue() does NOT exist in Appian */
a!dateTimeValue(year: year(today()), month: month(today()), day: 1)
```

**Rule:** Use `dateTime()` (lowercase T), NOT `a!dateTimeValue()` which does not exist.

---

## Date vs DateTime: Critical Distinction

**Date and DateTime are DIFFERENT types in Appian. They are NOT interchangeable.**

### Date Type
- **Represents:** Calendar date only (no time component)
- **Functions:** `today()`, `todate()`, `date()`, `datevalue()`, `eomonth()`, `edate()`
- **Arithmetic:** `today() + 30`, `today() - 7`
- **Use for:** Due dates, birth dates, start dates, any date-only field

### DateTime Type
- **Represents:** Date AND time (timestamp)
- **Functions:** `now()`, `todatetime()`, `dateTime()`, `a!addDateTime()`, `a!subtractDateTime()`, `userdatetime()`
- **Arithmetic:** `a!addDateTime()`, `a!subtractDateTime()` (NOT + or -)
- **Use for:** Created timestamps, modified timestamps, event times

---

## Date/DateTime Arithmetic

### Date Arithmetic (Simple Addition/Subtraction)

```sail
/* ✅ Date fields - use + and - */
today() + 30              /* Returns: Date (30 days from today) */
today() - 7               /* Returns: Date (7 days ago) */
local!dueDate + 14        /* Returns: Date (14 days from due date) */

/* ✅ Use in calculations */
if(
  a!isNotNullOrEmpty(local!startDate),
  todate(local!startDate + 365),
  null
)
```

### DateTime Arithmetic (Use a!addDateTime/a!subtractDateTime)

```sail
/* ✅ DateTime fields - use a!addDateTime() and a!subtractDateTime() */
a!addDateTime(startDateTime: now(), days: 30)       /* 30 days from now */
a!subtractDateTime(startDateTime: now(), days: 7)   /* 7 days ago */

/* ❌ WRONG - Cannot use + or - with DateTime */
now() + 30  /* ERROR: Returns Interval, not DateTime */
```

### Interval Return Type

**Subtracting dates or datetimes returns an Interval type, not a Number:**

```sail
/* Date subtraction returns Interval (Day to Day) */
today() - local!startDate  /* Returns: Interval */

/* DateTime subtraction returns Interval (Day to Second) */
now() - local!timestamp  /* Returns: Interval */

/* ❌ WRONG - Cannot compare Interval directly to Number */
if(now() - local!timestamp < 1, ...)  /* ERROR */

/* ✅ CORRECT - Convert Interval to Integer first */
if(tointeger(now() - local!timestamp) < 1, ...)  /* ✓ Works */
```

---

## Type Compatibility for Query Filters

**When used in query filters (a!queryFilter), field type and value type MUST match exactly.**

### Common Mismatches

```sail
/* ❌ WRONG - Date arithmetic with DateTime field */
a!queryFilter(
  field: recordType!Case.fields.createdOn,  /* DateTime field */
  value: today() - 30  /* Date value - TYPE MISMATCH */
)

/* ✅ CORRECT - Use DateTime functions for DateTime fields */
a!queryFilter(
  field: recordType!Case.fields.createdOn,  /* DateTime field */
  value: a!subtractDateTime(startDateTime: now(), days: 30)  /* DateTime value */
)

/* ✅ CORRECT - Date field with Date value */
a!queryFilter(
  field: recordType!Case.fields.dueDate,  /* Date field */
  value: today() - 30  /* Date value */
)
```

### Type Conversion Between Date and DateTime

```sail
/* Convert Date to DateTime */
todatetime(today())  /* Today at midnight */

/* Convert DateTime to Date */
todate(now())  /* Current date (drops time) */

/* Example: Using Date value with DateTime field */
a!queryFilter(
  field: recordType!Case.fields.createdOn,  /* DateTime field */
  value: todatetime(today() - 30)  /* Convert Date to DateTime */
)

/* Example: Using DateTime value with Date field */
a!queryFilter(
  field: recordType!Case.fields.dueDate,  /* Date field */
  value: todate(now())  /* Convert DateTime to Date */
)
```

---

## Function Return Types Quick Reference

### Date Functions

| Function | Returns | Use For |
|----------|---------|---------|
| `today()` | Date | Current date only |
| `todate(...)` | Date | Convert value to Date |
| `date(y, m, d)` | Date | Create specific Date |
| `datevalue(text)` | Date | Parse text string to Date |
| `eomonth(startDate, months)` | Date | Last day of month N months from startDate |
| `edate(startDate, months)` | Date | Same day N months from startDate |
| `today() + N` / `today() - N` | Date | Date arithmetic |

### DateTime Functions

| Function | Returns | Use For |
|----------|---------|---------|
| `now()` | DateTime | Current date and time |
| `todatetime(...)` | DateTime | Convert value to DateTime |
| `dateTime(y, m, d, h, m, s)` | DateTime | Create specific DateTime |
| `a!subtractDateTime(...)` | DateTime | Past DateTime calculation |
| `a!addDateTime(...)` | DateTime | Future DateTime calculation |
| `userdatetime(y, m, d, h, m, s)` | DateTime | Create DateTime in user's preferred calendar |

### Other Type Functions

| Function | Returns | Use For |
|----------|---------|---------|
| `tointeger(...)` | Integer | Convert to whole number |
| `todecimal(...)` | Decimal | Convert to decimal number |
| `tostring(...)` | Text | Convert to text |
| `true()` / `false()` | Boolean | Boolean literals |
| `loggedInUser()` | User | Current user |
| `touser(...)` | User | Convert username/value to User |
| `togroup(...)` | Group | Convert group ID/value to Group |

### Formatting (Returns Text, not Date/DateTime)

| Function | Returns | Use For |
|----------|---------|---------|
| `text(value, format)` | Text | Format Date or DateTime as text string |

---

## Type Matching in Expression Rules

When passing date/time values in expression rules, the types must match the parameter definitions:

```sail
/* Expression rule with Date parameter */
rule!calculateDueDate(
  startDate: today(),           /* ✅ Date type */
  daysToAdd: 30
)

/* Expression rule with DateTime parameter */
rule!calculateTimestamp(
  startDateTime: now(),         /* ✅ DateTime type */
  daysToAdd: 30
)

/* ❌ WRONG - Type mismatch */
rule!calculateDueDate(
  startDate: now(),             /* ❌ DateTime passed to Date parameter */
  daysToAdd: 30
)

/* ✅ CORRECT - Convert type */
rule!calculateDueDate(
  startDate: todate(now()),     /* ✅ Convert DateTime to Date */
  daysToAdd: 30
)
```

---

## text() Function with Date/Time Values

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
```

**For complete text() patterns, see [null-safety-patterns.md](null-safety-patterns.md).**

---

## Complete Type Matching Reference

### Type Compatibility Matrix

| Field/Param Type | Compatible Value Types | Incompatible (Will Fail) |
|-----------------|----------------------|--------------------------|
| **Text** | Text literals, text variables, `tostring()`, `concat()` | Integer, Boolean, Date, DateTime, User |
| **Number (Integer)** | Integer literals, integer variables, `tointeger()`, Decimal | Text, Boolean, Date, DateTime |
| **Number (Decimal)** | Decimal literals, decimal variables, `todecimal()`, Integer | Text, Boolean, Date, DateTime |
| **Date** | `today()`, `todate()`, `date()`, `datevalue()`, `eomonth()`, `edate()`, date arithmetic, Date variables | DateTime, `now()`, `a!subtractDateTime()`, `a!addDateTime()`, `userdatetime()`, Text, Integer |
| **DateTime** | `now()`, `todatetime()`, `dateTime()`, `a!subtractDateTime()`, `a!addDateTime()`, `userdatetime()`, DateTime variables | Date, `today()`, `todate()`, `datevalue()`, `eomonth()`, `edate()`, Text, Integer |
| **Boolean** | `true()`, `false()`, Boolean variables | Text (`"true"`), Integer (`1`, `0`) |
| **User** | `loggedInUser()`, `touser()`, User variables | Text (username string without `touser()`) |
| **Group** | `togroup()`, Group variables | Integer (group ID without `togroup()`), Text |

**Note:** Integer and Decimal are the ONLY interchangeable types. All other combinations require exact match or explicit conversion.

---

## Date Function Corrections

```sail
/* ❌ WRONG - addDateTime rejects negative values */
value: a!addDateTime(startDateTime: today(), days: -30)

/* ✅ CORRECT - Use subtractDateTime for past dates */
value: a!subtractDateTime(startDateTime: now(), days: 30)

/* ❌ WRONG - Using date arithmetic with DateTime */
value: now() - 30

/* ✅ CORRECT - Use a!subtractDateTime for DateTime */
value: a!subtractDateTime(startDateTime: now(), days: 30)
```

---

## min()/max() Return Type Casting

**The min() and max() functions return variant types that may need explicit casting for comparisons:**

```sail
/* ❌ WRONG - min() result used directly without type casting */
local!startDates: local!courses.startDate,  /* Array of Date values */
local!earliestStart: min(local!startDates),  /* Returns variant */
local!isUrgent: local!earliestStart < today() + 30  /* May cause type error */

/* ✅ CORRECT - Cast min() result to proper type */
local!startDates: local!courses.startDate,  /* Array of Date values */
local!earliestStart: todate(min(local!startDates)),  /* Explicit Date type */
local!isUrgent: local!earliestStart < today() + 30  /* Date comparison works */
```

### Rules:
- **Date arrays**: Wrap with `todate(min(...))` or `todate(max(...))`
- **DateTime arrays**: Wrap with `todatetime(min(...))` or `todatetime(max(...))`
- **Number arrays**: Wrap with `tointeger(...)` or `todecimal(...)` if specific type needed
- **Always cast** when using the result in comparisons or calculations

---

## Common Patterns

### Pattern 1: Date Range Calculation

```sail
/* Calculate date range (Date type) */
local!startDate: today(),
local!endDate: today() + 30,
local!daysBetween: tointeger(local!endDate - local!startDate)
/* Returns: 30 */
```

### Pattern 2: DateTime Range Calculation

```sail
/* Calculate datetime range (DateTime type) */
local!startTime: now(),
local!endTime: a!addDateTime(startDateTime: now(), hours: 2),
local!hoursBetween: tointeger((local!endTime - local!startTime) / 3600)
/* Returns: 2 (Interval converted to seconds, then to hours) */
```

### Pattern 3: Age Calculation from Birth Date

```sail
/* Calculate age from birth date */
local!birthDate: date(1990, 5, 15),
local!age: year(today()) - year(local!birthDate)
/* Returns: Current age in years */
```

### Pattern 4: Business Days Calculation

```sail
/* Add business days (excluding weekends) */
rule!addBusinessDays(
  startDate: today(),
  businessDays: 5
)
/* Returns: Date 5 business days from today */
```

### Pattern 5: Month-End Calculations

```sail
/* Get last day of current month */
local!lastDayOfMonth: eomonth(today(), 0)

/* Get last day of next month */
local!lastDayNextMonth: eomonth(today(), 1)

/* Get last day of previous month */
local!lastDayPrevMonth: eomonth(today(), -1)
```

---

## Summary

**🚨 CRITICAL RULES:**
1. **a!dateTimeValue() does NOT exist** - Use `dateTime()` instead
2. **Date and DateTime are different types** - Cannot be used interchangeably
3. **Date arithmetic** - Use `+` and `-` for Date values
4. **DateTime arithmetic** - Use `a!addDateTime()` and `a!subtractDateTime()` for DateTime values
5. **Interval return type** - Date/DateTime subtraction returns Interval, convert with `tointeger()`
6. **Type matching** - Expression rule parameters and query filters require exact type match
7. **Type conversion** - Use `todate()` to convert DateTime to Date, `todatetime()` to convert Date to DateTime
8. **min()/max() casting** - Always wrap in type converter when used in comparisons
9. **text() null safety** - Check for null BEFORE calling text() (see null-safety-patterns.md)

**Key Functions:**
- **Date:** `today()`, `todate()`, `date()`, `eomonth()`, `edate()`, `datevalue()`
- **DateTime:** `now()`, `todatetime()`, `dateTime()`, `a!addDateTime()`, `a!subtractDateTime()`
- **Formatting:** `text(value, format)` - Always check for null first
- **Type casting:** `todate()`, `todatetime()`, `tointeger()` for Intervals
