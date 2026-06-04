---
name: "appian-expressions"
description: "Write Appian expressions for any context — process models, expression rules, interfaces, or conditions. Covers expression syntax, operators, the type system, object reference patterns, and common functions. Use when the agent needs to write expression syntax, reference Appian objects, use Appian functions, or decide between inline expressions and expression rules."
---

## Writing Expressions

All Appian expressions start with `=` when used in expression rule bodies, interface expressions, process model script tasks, and XOR gateway conditions. Expressions are strongly typed and follow function-call syntax: `functionName(param1, param2)`. Named parameters use colon syntax: `functionName(paramName: value)`.

### Operators

Use these operators in expressions:

- Arithmetic: `+`, `-`, `*`, `/` (division returns Decimal)
- Comparison: `=`, `<>`, `<`, `>`, `<=`, `>=`
- String concatenation: `&` (converts non-text operands to text automatically)
- Logical: `and()`, `or()`, `not()` — these are functions, not operators
- Negation: `-` (unary minus for numbers)
- List construction: `{item1, item2, item3}`
- List indexing: `myList[1]` (1-based indexing)
- Field access: `record[recordType!Entity.fields.fieldName]`

### Null Handling

Always guard against nulls. Appian propagates nulls through most operations — `null + 1` returns `null`, not `1`.

- Use `isnull(value)` to test for null
- Use `a!isNullOrEmpty(value)` to test for null, empty string, or empty list
- Use `a!defaultValue(value, default)` to provide a fallback when a value is null or empty
- Use `if(isnull(x), fallback, x)` for conditional null handling

### Comments

Use `/* comment */` for inline comments within expressions. There is no line-comment syntax.

## Type System

Appian is strongly typed. All variables have a declared type, and the platform auto-casts between compatible types where possible (e.g., Integer to Decimal in comparisons). When auto-casting is insufficient, use explicit casting functions.

### Primitive Types

`Text`, `Number (Integer)`, `Number (Decimal)`, `Boolean`, `Date`, `Time`, `Date and Time`, `User`, `Group`, `Document`, `Folder`

### Casting Functions

- `tointeger(value)` — convert to Integer (truncates decimals)
- `todecimal(value)` — convert to Decimal
- `tostring(value)` — convert to Text
- `toboolean(value)` — convert to Boolean
- `todate(value)` — convert to Date
- `todatetime(value)` — convert to Date and Time
- `totime(value)` — convert to Time
- `touser(value)` — convert to User (from username text)
- `togroup(value)` — convert to Group
- `cast(typeNumber, value)` — general-purpose cast using type number or `'type!{namespace}typeName'`
- `typeof(value)` — get the type number of a value
- `typename(typeNumber)` — get the type name from a type number

When casting between types, be aware that some casts lose information (Decimal 123.45 → Integer 123). Comparison operators auto-normalize types: Integer vs. Decimal promotes to Decimal; any type vs. Text promotes to Text.

### Record Types as Types

Record types are first-class types. Use `recordType!EntityName` to reference a record type as a type. Construct record instances with field syntax:

```
recordType!Employee(
  recordType!Employee.fields.firstName: "Jane",
  recordType!Employee.fields.lastName: "Doe"
)
```

Cast between records and maps: `cast(typeof(a!map()), myRecord)` or `cast(recordType!Employee, myMap)`.

## Object Reference Patterns

Appian uses domain-prefixed references to access different object types. Use the correct prefix for each context.

### Reference Prefixes

| Prefix | What It References | Example |
|---|---|---|
| `recordType!` | Record type and its fields/relationships | `recordType!Case.fields.status` |
| `cons!` | Application constant | `cons!STATUS_OPEN` |
| `rule!` | Expression rule | `rule!MYAPP_GetFullName(firstName: "Jane", lastName: "Doe")` |
| `ri!` | Rule input (parameter to current expression rule or interface) | `ri!caseRecord` |
| `pv!` | Process variable (in process model context) | `pv!isUpdate` |
| `rv!` | Record variable (in record action/view context) | `rv!record` |
| `fn!` | System function (rarely needed — most functions are called directly) | `fn!if(true, "yes", "no")` |
| `dp!` | Decision object | `dp!MyDecision` |
| `a!` | Appian function (components, utilities) | `a!localVariables()`, `a!forEach()` |

### Record Type References — UUID-Qualified Format

When writing expressions programmatically through the API (interface expressions, process model script tasks, XOR conditions, start form expressions, record view/action expressions), record type references must use the UUID-qualified format. The Appian expression parser cannot resolve plain-text names when expressions are submitted via the API — it needs the UUID embedded in curly braces before the display name.

**Format**: `'recordType!{recordTypeUuid}Display Name.fields.{fieldUuid}fieldName'`

The entire reference string must be wrapped in single quotes. This is required for the expression parser to treat the reference (including spaces and special characters) as a single token.

**Example** — referencing a record type and field:
```
'recordType!{7819d7d4-b730-442b-b27a-0408f61b29c3}Case.fields.{18248306-71fa-406f-be57-718e1ed5c6a2}status'
```

**Example** — referencing a relationship then a field on the related type:
```
'recordType!{7819d7d4-b730-442b-b27a-0408f61b29c3}Case.relationships.{rel-uuid}customer.fields.{field-uuid}name'
```

**Example** — in a SAIL expression context:
```
local!record['recordType!{7819d7d4-b730-442b-b27a-0408f61b29c3}Case.fields.{18248306-71fa-406f-be57-718e1ed5c6a2}status']
```

This applies everywhere a `recordType!` reference appears in an expression submitted through the API:
- `updateInterfaceExpression` — SAIL expressions
- `createProcessModel` / `updateProcessModel` — Script Task `outputVariables` expressions, XOR `condition` expressions, `startFormExpr`
- `addRecordTypeView` — `interfaceExpression`
- `addRecordTypeAction` — `contextExpr`, `visibilityExpr`
- `createExpressionRule` / `updateExpressionRule` — `expression` body

To build UUID-qualified references, retrieve the record type UUID from `createRecordType` or `getRecordType`, and field UUIDs from `listRecordTypeFields` or the `sourceAndCustomFields` in the create response.

**Plain-text shorthand** (`recordType!Case.fields.status`) is shown in examples throughout this skill for readability. When generating actual expressions for the API, always substitute the UUID-qualified format.

### Record Type Field Access

Access record fields using bracket notation with the full field path:

```
rv!record[recordType!Case.fields.status]
rv!record[recordType!Case.fields.assignedTo]
```

For related records (via relationships):

```
rv!record[recordType!Case.relationships.customer][recordType!Customer.fields.name]
```

### Process Variable References

In process model script tasks and XOR conditions, reference process variables with `pv!`:

```
pv!caseRecord
pv!isUpdate
pv!cancel
```

### Rule Input References

In expression rules and interfaces, reference inputs with `ri!`:

```
ri!caseRecord
ri!searchText
```

The `ri!` name must exactly match the rule input name defined on the expression rule or interface.

### Record Variable References

In record action security expressions and record views, use `rv!record` to reference the current record:

```
rv!record[recordType!Case.fields.status] = cons!STATUS_OPEN
```

## Inline Expressions vs. Expression Rules

Choose the right approach based on reuse and complexity:

- Use inline expressions for simple, one-off logic: XOR conditions, script task assignments, visibility conditions, single-use calculations
- Create expression rules when logic is reused across multiple objects, when the logic is complex enough to benefit from a descriptive name, or when the logic needs independent testing
- Expression rules are called with `rule!` prefix and accept typed inputs via `ri!` parameters
- Name expression rules with an application prefix: `rule!MYAPP_CalculateRiskScore(score: ri!creditScore)`

## Common Patterns

### Security Expressions

Build security expressions for record actions and page visibility using these patterns:

```
/* Role-based check */
a!isUserMemberOfGroup(loggedInUser(), cons!CASE_MANAGER_GROUP)

/* Status-based check */
rv!record[recordType!Case.fields.status] = cons!STATUS_OPEN

/* Combined role + status check */
and(
  a!isUserMemberOfGroup(loggedInUser(), cons!CASE_MANAGER_GROUP),
  or(
    rv!record[recordType!Case.fields.status] = cons!STATUS_OPEN,
    rv!record[recordType!Case.fields.status] = cons!STATUS_IN_PROGRESS
  )
)
```

### Process Model Script Task Expressions

In script task `expressionBody` and `outputVariables`, write expressions that compute values and assign them to process variables:

```
/* Conditional assignment */
if(pv!isUpdate, "Updated", "Created")

/* Record field extraction */
pv!caseRecord[recordType!Case.fields.assignedTo]
```

### XOR Gateway Conditions

XOR conditions evaluate to Boolean — the first true condition determines the path:

```
pv!isUpdate = true
pv!caseRecord[recordType!Case.fields.priority] = "High"
```

### Local Variables

Use `a!localVariables()` to define intermediate values in complex expressions:

```
a!localVariables(
  local!fullName: ri!firstName & " " & ri!lastName,
  local!isValid: and(
    a!isNotNullOrEmpty(ri!firstName),
    a!isNotNullOrEmpty(ri!lastName)
  ),
  if(local!isValid, local!fullName, "Unknown")
)
```

### Iterating Over Lists

Use `a!forEach()` to transform lists. Inside the expression, use function variables:

- `fv!item` — the current item
- `fv!index` — the current 1-based index
- `fv!isFirst`, `fv!isLast` — position flags
- `fv!itemCount` — total items

```
a!forEach(
  items: ri!cases,
  expression: fv!item[recordType!Case.fields.title]
)
```

## Pitfalls

- Using plain-text record type names in API expressions — `recordType!Board Committee Submission.fields.status` will fail because the parser can't resolve names with spaces (or any names) without UUIDs; always use the UUID-qualified format `'recordType!{uuid}Name.fields.{fieldUuid}fieldName'` (with single quotes wrapping the entire reference) when writing expressions through the API
- Forgetting that `ri!` names must exactly match the defined rule input or interface input name
- Using `=` for null comparison — use `isnull()` instead, since `null = null` returns `null`, not `true`
- Assuming 0-based indexing — Appian lists are 1-based
- Dividing integers and expecting an integer result — `/` always returns Decimal
- Concatenating non-text values without `&` — use `&` which auto-converts, or explicitly cast with `tostring()`
- Nesting `if()` deeply — prefer `choose()` or `a!match()` for multi-branch logic
- Forgetting that `a!forEach()` returns a list — even for a single item, the result is `{value}` not `value`

## When You Need More

For questions about specific Appian functions, edge cases, or advanced expression patterns beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.

Load `references/common-functions.md` when you need function signatures and examples for logical, text, date/time, list, or record type functions.
