# Expression Rules and Patterns Reference

This reference provides architectural guidance for expression rules: management patterns (create vs update vs version), common use cases, performance considerations, and best practices.

**⚠️ IMPORTANT: Before writing ANY expression, load [function-reference.md](function-reference.md) for:**
- Anti-hallucination list (regexmatch doesn't exist!)
- Function signatures and parameters
- Working code examples (email validation, queries)
- Appian-specific gotchas (find() is 1-indexed, union() validator workaround)

---

## Table of Contents

**Fundamentals**
- [Writing Expressions](#writing-expressions)
- [Type System](#type-system)
- [Object Reference Patterns](#object-reference-patterns)
- [Inline Expressions vs. Expression Rules](#inline-expressions-vs-expression-rules)

**Creating Expression Rules**
- [Gathering Expression Rule Requirements](#gathering-expression-rule-requirements)

**Expression Rule Management**
- [Expression Rule Management](#expression-rule-management) (overview)
  - [When to Create vs Update](#when-to-create-vs-update)
  - [Input Design Patterns](#input-design-patterns)
  - [Breaking Change Detection](#breaking-change-detection)

**Common Patterns**
- [Common Patterns](#common-patterns) (overview)
  - [Record Type Queries](#record-type-queries)
  - [Data Transformation](#data-transformation)
  - [Validation Rules](#validation-rules)
  - [Calculated Fields](#calculated-fields)
  - [Error Handling Patterns](#error-handling-patterns)

**Confirmation Patterns**
- [Confirmation Patterns](#confirmation-patterns)

**Pitfalls and Best Practices**
- [Common Pitfalls](#common-pitfalls)
- [Best Practices](#best-practices)

**Additional Resources**
- [When You Need More](#when-you-need-more)

---

<a name="writing-expressions"></a>
## Writing Expressions

Expression rule bodies must start with `=`. Within expressions (like in `a!localVariables()`, interface component properties, or process model script tasks), the `=` is NOT used — assignments use colon syntax: `local!name: value` or `paramName: value`. Expressions are strongly typed and follow function-call syntax: `functionName(param1, param2)`.

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

- Use `a!isNullOrEmpty(value)` to test for null, empty string, or empty list (preferred)
- Use `a!isNotNullOrEmpty(value)` to test for non-null and non-empty values
- Use `a!defaultValue(value, default)` to provide a fallback when a value is null or empty
- Use `if(a!isNullOrEmpty(x), fallback, x)` for conditional null handling

### Comments

Use `/* comment */` for inline comments within expressions. There is no line-comment syntax.

<a name="type-system"></a>
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

<a name="object-reference-patterns"></a>
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

When writing expressions programmatically (interface expressions, process model script tasks, XOR conditions, start form expressions, record view/action expressions), record type references must use the UUID-qualified format. The Appian expression parser cannot resolve plain-text names — it needs the UUID embedded in curly braces before the display name.

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

This applies everywhere a `recordType!` reference appears in an expression:
- Interface expressions
- Process model expressions (Script Task outputs, XOR conditions, start form)
- Record type view interface expressions
- Record action context and visibility expressions
- Expression rule bodies

To build UUID-qualified references, retrieve the record type UUID from the creation or get response, and field UUIDs from the fields list or the creation response.

**Plain-text shorthand** (`recordType!Case.fields.status`) is shown in examples throughout this skill for readability. When generating actual expressions, always substitute the UUID-qualified format.

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

<a name="inline-expressions-vs-expression-rules"></a>
## Inline Expressions vs. Expression Rules

Choose the right approach based on reuse and complexity:

- Use inline expressions for simple, one-off logic: XOR conditions, script task assignments, visibility conditions, single-use calculations
- Create expression rules when logic is reused across multiple objects, when the logic is complex enough to benefit from a descriptive name, or when the logic needs independent testing
- Expression rules are called with `rule!` prefix and accept typed inputs via `ri!` parameters
- Name expression rules with an application prefix: `rule!MYAPP_CalculateRiskScore(score: ri!creditScore)`

<a name="gathering-expression-rule-requirements"></a>
## Gathering Expression Rule Requirements

**CRITICAL:** Before creating an expression rule, ensure you have complete requirements. Clarifying is cheaper than rework.

### When to Ask for Clarification

**ALWAYS ask the user for details when the request is missing:**
- **Input specification** - What data goes in? (names, types, required vs optional)
- **Output specification** - What comes out? (type, format, structure)
- **Business logic** - What are the rules? What validation is needed?
- **Edge cases** - How to handle null, empty, invalid data?
- **Scope** - Basic implementation or production-grade with comprehensive validation?

**Critical principle:** Even when reasonable defaults exist (e.g., standard query patterns), **always confirm** input parameters, returned fields, and business logic before implementing. The cost of asking is low; the cost of mismatched expectations is high.

---

### Common Ambiguous Requests

| Request | Missing Information | What to Ask |
|---------|-------------------|-------------|
| "Create a validation rule" | Return type, validation rules | "Should this return: (1) Boolean true/false, (2) Error message text, or (3) Validation map with isValid + errors? What fields need validation and what are the rules?" |
| "Query cases" | Fields, filters, sorting, batch size | "Which fields should be returned? Any specific filters? Should results be sorted? How many records (batch size)?" |
| "Calculate total" | What to sum, filters, null handling | "Calculate total of which field? Should any records be excluded? How should null values be handled?" |
| "Email validation" | Validation depth, return type | "Should this be: (1) Basic validation (check for @), or (2) Production-grade (format, length, character validation)? Return boolean or error message?" |
| "Format date" | Format string, timezone, null handling | "What date format? (e.g., 'MM/DD/YYYY', 'YYYY-MM-DD')? Should timezone be considered? How to handle null dates?" |
| "Transform data" | Specific transformation logic | "What transformation is needed? Can you provide an example of input → expected output?" |

---

### Requirements Checklist

Before creating an expression rule, gather these details:

**1. Inputs**
- [ ] Input names (follow camelCase convention)
- [ ] Input types (Text, Number, Boolean, User, Record, Date, etc.)
- [ ] Which inputs are required vs optional?
- [ ] Default values for optional inputs (if any)
- [ ] Expected ranges or constraints (e.g., "score between 0-100")

**Example questions:**
```
"What inputs does this rule need? For each input:
- Name?
- Type? (Text, Number, Boolean, etc.)
- Required or optional?
- Any validation constraints?"
```

**2. Output**
- [ ] Return type (Text, Number, Boolean, Record, Array, Map, etc.)
- [ ] Return format (for Text: is there a specific format? for Number: integer or decimal?)
- [ ] For validation rules: Boolean, error message (Text), or validation map?
- [ ] For queries: Array of records with specific fields, or full records?

**Example questions:**
```
"What should this rule return?
- Boolean true/false?
- Error message text?
- Validation map with {isValid: Boolean, errors: Text array}?
- Record data? If so, which fields?"
```

**3. Business Logic**
- [ ] What are the rules/conditions?
- [ ] How to handle edge cases? (null, empty, invalid data)
- [ ] Are there multiple validation rules? (email + date range + required fields)
- [ ] Performance considerations? (large data sets, complex queries)

**Example questions:**
```
"What are the validation rules?
- Email: required? valid format? max length?
- Dates: range validation? start before end?
- Text fields: max length? allowed characters?

How should edge cases be handled?
- Null or empty inputs?
- Invalid data?"
```

**4. Scope**
- [ ] Basic implementation or production-grade?
- [ ] Related record data needed? (for queries)
- [ ] Sorting/paging required? (for queries)
- [ ] Should results include specific related fields? (e.g., status.label, priority.label)

**Example questions:**
```
"Should this be:
1. Basic validation (minimal checks)?
2. Production-grade validation (comprehensive checks)?

For queries:
- Which record type fields should be returned?
- Include related record data? (e.g., status label, priority label)
- Sorting needed? If so, by which field?
- Batch size? (default: 100 for queries)"
```

---

### Example: Clarifying a Validation Request

**User request:**
> "Create a validation rule for email"

**Ambiguities:**
- Return type? (Boolean vs error message vs validation map)
- Validation depth? (Basic @ check vs production-grade format validation)
- Input type? (Single email vs array of emails)

**Good clarification response:**
```
I'll create an email validation rule. A few questions to ensure I build what you need:

1. Return type - should this return:
   • Boolean (true if valid, false if invalid)
   • Error message text (null if valid, message if invalid)
   • Validation map ({isValid: Boolean, errors: Text array})

2. Validation depth:
   • Basic: Check for @ symbol and non-empty
   • Production-grade: Check format, max length (255), no spaces, valid characters, domain structure

3. Input:
   • Single email (Text)
   • Array of emails (Text array)

Recommendation: Production-grade validation returning Boolean for reusable logic.
Is this correct?
```

---

### Example: Clarifying a Query Request

**User request:**
> "Query cases assigned to a user"

**Ambiguities:**
- Which fields to return?
- Filter by status/priority?
- Include related record data?
- Sorting needed?
- Batch size?

**Good clarification response:**
```
I'll create a query rule for cases. To build this correctly, I need to know:

1. Which Case fields should be returned?
   • All fields?
   • Specific fields? (e.g., id, title, description, status, priority)

2. Filters beyond "assigned to user":
   • Filter by status? (e.g., "Open" or "In Progress")
   • Filter by priority? (e.g., "High" or "Critical")

3. Related record data:
   • Include status label?
   • Include priority label?
   • Any other related fields?

4. Sorting:
   • Sort by creation date (newest first)?
   • Sort by another field?

5. Batch size:
   • Default 100 records?
   • Different size needed?

Recommendation based on common pattern:
- Fields: id, title, description, assignedTo, createdAt
- Related: status.id, status.label, priority.id, priority.label
- Sort: createdAt descending
- Batch: 100

Does this match what you need?
```

---

### Example: Clarifying "Get Cases by Priority"

**User request:**
> "Get cases by priority"

**Why this needs clarification despite having standard query patterns:**
Even though queries have conventions (batch size 100, sort by createdAt, etc.), the core requirements are ambiguous.

**Good clarification response:**
```
I'll create a rule to get cases by priority. To ensure I build what you need:

1. Priority input parameter:
   • priorityId (Number) - filter by priority ID?
   • priorityLabel (Text) - filter by priority label like "High", "Critical"?
   • Both options available?

2. Which Case fields should be returned?
   • All fields (id, title, description, dates, assigned user, etc.)?
   • Specific subset? (e.g., just id + title + priority)

3. Include related data?
   • Status information (id + label)?
   • Assigned user information?

4. Any additional filters beyond priority?
   • Only open cases?
   • Only assigned cases?

Recommendation: 
- Input: Both priorityId and priorityLabel (OR logic, flexible)
- Fields: All case fields
- Related: status.id, status.label, priority.id, priority.label
- Sort: createdAt descending (standard)
- Batch: 100 (standard)

Does this match your needs?
```

**Why not assume "reasonable defaults":**
- User might want simpler output (just id + title) for performance
- User might want single input type (ID only or label only)
- User might want additional filters (status = "Open")
- Confirming prevents rework when assumptions don't match user's mental model

---

### Anti-Patterns: Don't Make These Assumptions

**❌ WRONG: Assuming validation depth**
```
User: "Create email validation"
AI: [Creates basic @ check without asking]
Result: User expected production-grade validation with format rules
```

**❌ WRONG: Assuming return type**
```
User: "Create validation for case form"
AI: [Returns Boolean without asking]
Result: User needed error messages for display in form
```

**❌ WRONG: Assuming query scope**
```
User: "Query cases"
AI: [Returns all fields + all related data without asking]
Result: Performance issue; user only needed id + title
```

**✅ CORRECT: Ask first, build second**
```
User: "Create email validation"
AI: "Should this return Boolean or error message? Basic validation or production-grade?"
User: "Production-grade, return Boolean"
AI: [Creates comprehensive validation with correct return type]
Result: Matches user's mental model on first attempt
```

---

### When NOT to Ask (Apply Defaults Silently)

Some details have standard conventions and don't need clarification:

**Naming conventions:**
- Use application prefix automatically (CM_ for Case Management app)
- Use camelCase for inputs (no need to ask)
- Use descriptive verb+noun names (ValidateEmail, QueryCases, CalculateTotal)

**Query implementation details (AFTER confirming inputs/fields/filters):**
- Batch size 100 for queries
- Sort queries by creation date descending
- Return `.data` from queries (array of records)
- Include id + label format for related lookup tables (status.id, status.label)

**Default behaviors:**
- Trim text inputs before validation
- Use `a!defaultValue()` for optional inputs
- Apply null safety patterns automatically

**What you MUST still ask about (even when standard patterns exist):**
- **Input parameters** - Even for standard queries, confirm: "Should priority be a rule input? If so, priorityId (Number) or priorityLabel (Text)?"
- **Returned fields** - Even for standard queries, confirm: "Which Case fields should be returned? All fields or specific ones?"
- **Filters** - Even for standard queries, confirm: "Any additional filters beyond priority (e.g., status = 'Open')?"
- **Return type for validations** - Boolean, error message, or validation map?
- **Validation depth** - Basic or production-grade?

**Principle:** 
- Clarify **behavior and requirements** (inputs, outputs, business logic), not **conventions** (naming, null safety, standard patterns)
- When in doubt between asking and assuming → **ask**
- Standard patterns are applied AFTER requirements are confirmed, not instead of confirmation

---

## Before Implementing Custom Logic

**MANDATORY: Check if built-in function exists before writing custom logic**

If your expression requires any of these operations, search Appian docs FIRST:

- **Distance/geographic calculations** → search "distance"
- **Encryption/hashing** → search "encrypt", "hash"  
- **Date arithmetic (working days, business days)** → search "workday", "networkday"
- **Data transformations (JSON, XML, CSV)** → search "json", "xml", "csv"
- **String parsing beyond basic functions** → search relevant keywords
- **Mathematical operations beyond basic math** → search "math", function name

### How to Search for Functions

**Step 1: Search functions.json by keyword**

```bash
# Read configured version from SKILL.md Configuration section
VERSION="26.6"

# Cache functions.json if not already cached
if [ ! -f /tmp/appian-functions-$VERSION.json ]; then
  curl -s "https://docs.appian.com/suite/help/$VERSION/functions.json" \
    > /tmp/appian-functions-$VERSION.json
fi

# Search for function by keyword (case-insensitive)
jq -r 'keys[] | select(test("distance|encrypt|workday"; "i"))' \
  /tmp/appian-functions-$VERSION.json
```

**Step 2: If matches found, fetch documentation**

See SKILL.md "Appian Documentation Search" section for complete workflow to:
- Fetch the documentation page
- Extract function signature and parameters
- Use in your expression

**Step 3: Use built-in if exists, implement custom if not**

### Why This Matters

**Built-in functions:**
- ✅ Battle-tested by Appian
- ✅ Optimized for performance
- ✅ Handle edge cases correctly
- ✅ Maintained by platform
- ✅ Shorter, cleaner code

**Custom implementations:**
- ❌ More complex and error-prone
- ❌ Harder to maintain
- ❌ May miss edge cases
- ❌ Untested in production
- ❌ Requires deep domain knowledge

**Example:** Calculating distance between coordinates:
- Built-in: `a!distanceBetween(startLat, startLon, endLat, endLon, "MILES")` (1 line)
- Custom: Haversine formula implementation (~40 lines with trig, conversions, validation)

---

<a name="expression-rule-management"></a>
## Expression Rule Management

Expression rules encapsulate reusable logic. Understanding when and how to manage them is critical for maintainable applications.

<a name="when-to-create-vs-update"></a>
### When to Create vs Update

**Create a new expression rule when:**
- Logic will be reused across multiple interfaces, process models, or other rules
- Complexity warrants a descriptive name for clarity
- Logic needs independent testing or documentation
- Encapsulating a calculation, transformation, validation, or query

**Update an existing expression rule when:**
- Fixing a bug in the logic
- Adding non-breaking functionality (new optional input with default)
- Improving performance without changing behavior
- Clarifying variable names or expression structure

**Create a NEW rule (don't update) when:**
- Changing behavior would break existing callers
- Removing or changing required inputs (breaking change)
- Changing return type (breaking change)
- Fundamentally different purpose (even if similar name)

### Creation and Update Workflow

**When creating or updating expression rules, follow Step 7 workflow from SKILL.md:**

**Step 7A: Generate Expression**
- Write the expression body as a string
- Ensure all `ri!` references match input parameter names
- Apply null safety patterns from null-safety-patterns.md
- Use only verified functions (complete Step 4A first)

**Step 7B: Validate Expression (MANDATORY)**
- Load `references/validation-checkpoint.md` for complete workflow
- Call `validateExpression` MCP tool with generated expression
- If validation fails → Fix errors and retry (up to 3 attempts)
- Common errors: syntax issues, unknown `ri!` names, type mismatches

**Step 7C: Create or Update (Only After Validation Passes)**
- ✅ Validation passed → Call `createExpressionRule` or `updateExpressionRule`
- Include validated expression, inputs array, and metadata

**Important:** Always validate expressions before creating or updating. This catches errors early and enables automatic retry, reducing failed object creations.

<a name="input-design-patterns"></a>
### Input Design Patterns

**Required vs Optional Inputs:**

Use required inputs for data that MUST be provided:
```
Rule: CM_GetFullName
Inputs: firstName (Text, required), lastName (Text, required)
Expression: =ri!firstName & " " & ri!lastName
```

Use optional inputs with defaults for optional behavior:
```
Rule: CM_GetFullName
Inputs: 
  - firstName (Text, required)
  - lastName (Text, required)
  - includeMiddle (Boolean, optional)
  - middleName (Text, optional)
Expression: 
  =if(
    a!defaultValue(ri!includeMiddle, false),
    ri!firstName & " " & a!defaultValue(ri!middleName, "") & " " & ri!lastName,
    ri!firstName & " " & ri!lastName
  )
```

**Input naming conventions:**
- `camelCase` for all inputs: `firstName`, `caseRecord`, `includeArchived`
- Boolean inputs use question-style: `isUpdate`, `includeArchived`, `shouldNotify`
- Record inputs use `record` suffix: `caseRecord`, `customerRecord`
- List inputs use plural: `cases`, `usernames`, `priorities`

**Input ordering:**
1. Required inputs first
2. Optional inputs last
3. Most commonly used inputs first within each group

### Output Typing

Expression rules return typed values. Be explicit about return type for clarity.

**Single-value returns:**
```
Rule: CM_CalculateRiskScore
Returns: Number (Integer) between 0-100
Expression: =if(ri!score < 0, 0, if(ri!score > 100, 100, ri!score))
```

**Record returns:**
```
Rule: CM_BuildCustomerRecord
Returns: recordType!Customer
Expression: 
  =recordType!Customer(
    recordType!Customer.fields.name: ri!name,
    recordType!Customer.fields.email: ri!email
  )
```

**List returns:**
```
Rule: CM_GetActiveUsers
Returns: List of User
Expression: =a!queryRecordType(...).data[recordType!User.fields.username]
```

**Null handling in returns:**
- Document when rule can return null
- Use `a!defaultValue()` to avoid nulls when possible
- For aggregations, prefer empty list `{}` over null

### Testing Strategies

**Test expression rules independently:**

1. **Unit test approach** — Test rule in isolation:
   - Use `testRule` MCP tool with sample inputs
   - Verify output matches expected for various input scenarios
   - Test edge cases: nulls, empty lists, boundary values

2. **Integration test approach** — Test in context:
   - Create test interface that calls rule with realistic data
   - Use `testInterface` to verify rule behaves correctly in UI context
   - Verify rule works with actual record type data

**Test cases to cover:**
- Happy path (valid inputs, expected output)
- Null inputs (if rule should handle nulls)
- Empty lists (if rule accepts lists)
- Boundary values (0, negative numbers, very large values)
- Invalid data (if rule should validate)

**Example test scenarios:**
```
Rule: CM_GetFullName(firstName, lastName)

Test cases:
✓ firstName="Jane", lastName="Doe" → "Jane Doe"
✓ firstName="", lastName="Doe" → " Doe" (or error if validation added)
✓ firstName=null, lastName="Doe" → null (or "Doe" if defaultValue used)
✓ firstName="Jane", lastName=null → "Jane " (or "Jane" if trimmed)
```

### Versioning Approaches

**When you need to change a rule with breaking changes:**

**Approach 1: Create new version, deprecate old**
```
Old rule: CM_GetFullName(firstName, lastName)
New rule: CM_GetFullName_v2(firstName, lastName, includeMiddle, middleName)

Migration path:
1. Create CM_GetFullName_v2 with new signature
2. Update callers gradually to use v2
3. Mark CM_GetFullName as deprecated (in description)
4. Delete CM_GetFullName when all callers migrated
```

**Approach 2: Add new optional inputs (non-breaking)**
```
Original: CM_GetFullName(firstName, lastName)
Enhanced: CM_GetFullName(firstName, lastName, includeMiddle, middleName)
         - includeMiddle defaults to false
         - middleName defaults to null

Result: Existing callers continue working (non-breaking)
```

**Approach 3: Create new rule with descriptive name**
```
Old: CM_GetFullName(firstName, lastName)
New: CM_GetFullNameWithMiddle(firstName, lastName, middleName)

Result: Both rules coexist, no migration needed
```

**Recommendation:** Prefer Approach 2 (non-breaking additions) when possible. Use Approach 1 when signature change is fundamental.

---

<a name="common-patterns"></a>
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

<a name="record-type-queries"></a>
### Record Type Queries

Query record types using `a!queryRecordType()` to fetch data:

**Basic query pattern:**
```
a!queryRecordType(
  recordType: recordType!Case,
  fields: {
    recordType!Case.fields.title,
    recordType!Case.fields.status,
    recordType!Case.fields.assignedTo
  },
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 10)
)
```

**Query with filters:**
```
a!queryRecordType(
  recordType: recordType!Case,
  filters: a!queryFilter(
    field: recordType!Case.fields.status,
    operator: "=",
    value: cons!STATUS_OPEN
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 50)
)
```

**Multiple filters (AND):**
```
a!queryRecordType(
  recordType: recordType!Case,
  filters: {
    a!queryFilter(
      field: recordType!Case.fields.status,
      operator: "=",
      value: cons!STATUS_OPEN
    ),
    a!queryFilter(
      field: recordType!Case.fields.assignedTo,
      operator: "=",
      value: loggedInUser()
    )
  },
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 50)
)
```

**Query with relationship traversal:**
```
a!queryRecordType(
  recordType: recordType!Case,
  fields: {
    recordType!Case.fields.title,
    recordType!Case.relationships.customer.fields.name
  },
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 10)
)
```

**Query result structure:**
```
/* Result has .data and .totalCount */
local!result: a!queryRecordType(...),
local!cases: local!result.data,
local!count: local!result.totalCount
```

<a name="data-transformation"></a>
### Data Transformation

Transform data between types and structures:

**List transformation:**
```
/* Extract field values from record list */
a!forEach(
  items: ri!cases,
  expression: fv!item[recordType!Case.fields.assignedTo]
)

/* Filter list based on condition */
where(
  a!forEach(
    items: ri!cases,
    expression: fv!item[recordType!Case.fields.status] = cons!STATUS_OPEN
  ),
  ri!cases
)

/* Build lookup map from list */
a!forEach(
  items: ri!statuses,
  expression: a!map(
    id: fv!item[recordType!Status.fields.id],
    label: fv!item[recordType!Status.fields.label]
  )
)
```

**Record to map conversion:**
```
cast(typeof(a!map()), ri!caseRecord)
```

**Map to record conversion:**
```
cast(recordType!Case, ri!caseMap)
```

**Flattening nested lists:**
```
flatten({
  ri!list1,
  ri!list2,
  ri!list3
})
```

**Removing duplicates:**
```
union(ri!listWithDuplicates, ri!listWithDuplicates)
```

<a name="validation-rules"></a>
### Validation Rules

Build validation expressions for user input.

**⚠️ CRITICAL: Before writing validation logic:**
1. **Load [function-reference.md](function-reference.md)** - Check anti-hallucination list (regexmatch doesn't exist!)
2. **Use the email validation pattern from function-reference.md** - It's tested and working (50 lines of code)
3. **Follow these patterns EXACTLY** - Do not improvise or modify

**Do NOT:**
- Generate your own validation logic from training data or general knowledge
- Modify these patterns unless the user explicitly requests different behavior
- Invent alternative validation approaches
- Use functions that are not shown in these examples
- Try to use regexmatch(), regex(), or any regex functions (they don't exist in Appian)

**These patterns are tested against the actual Appian environment and use only available functions. Follow them precisely.**

**If a function in these patterns doesn't work (e.g., validation error), tell the user immediately and ask for guidance. Do not improvise a replacement.**

---

**Required field validation:**
```
a!localVariables(
  local!isValid: and(
    a!isNotNullOrEmpty(ri!firstName),
    a!isNotNullOrEmpty(ri!lastName),
    a!isNotNullOrEmpty(ri!email)
  ),
  local!errors: if(
    local!isValid,
    null,
    {
      if(a!isNullOrEmpty(ri!firstName), "First name is required", null),
      if(a!isNullOrEmpty(ri!lastName), "Last name is required", null),
      if(a!isNullOrEmpty(ri!email), "Email is required", null)
    }
  ),
  a!map(
    isValid: local!isValid,
    errors: reject(fn!isnull, local!errors)
  )
)
```

**Email format validation:**
```
a!localVariables(
  local!emailPattern: "^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$",
  local!isValidEmail: regexmatch(local!emailPattern, ri!email),
  if(
    local!isValidEmail,
    null,
    "Invalid email format"
  )
)
```

**Range validation:**
```
a!localVariables(
  local!isInRange: and(
    ri!score >= 0,
    ri!score <= 100
  ),
  if(
    local!isInRange,
    null,
    "Score must be between 0 and 100"
  )
)
```

**Date range validation:**
```
a!localVariables(
  local!isValidRange: ri!endDate > ri!startDate,
  if(
    local!isValidRange,
    null,
    "End date must be after start date"
  )
)
```

<a name="calculated-fields"></a>
### Calculated Fields

Compute derived values from record data:

**Duration calculation:**
```
/* Days between dates */
a!localVariables(
  local!startDate: ri!caseRecord[recordType!Case.fields.createdAt],
  local!endDate: ri!caseRecord[recordType!Case.fields.closedAt],
  a!defaultValue(
    daysbetween(local!startDate, local!endDate),
    0
  )
)
```

**Aggregation across related records:**
```
/* Count related tasks */
a!localVariables(
  local!tasks: ri!projectRecord[recordType!Project.relationships.tasks],
  count(local!tasks)
)

/* Sum of amounts */
a!localVariables(
  local!expenses: ri!projectRecord[recordType!Project.relationships.expenses],
  sum(
    a!forEach(
      items: local!expenses,
      expression: fv!item[recordType!Expense.fields.amount]
    )
  )
)
```

**Conditional formatting:**
```
/* Risk level based on score */
a!localVariables(
  local!score: ri!caseRecord[recordType!Case.fields.riskScore],
  if(
    local!score >= 80,
    "HIGH",
    if(
      local!score >= 50,
      "MEDIUM",
      "LOW"
    )
  )
)
```

**Status badge color:**
```
a!match(
  value: rv!record[recordType!Case.fields.status],
  equals: cons!STATUS_OPEN,
  then: "BLUE",
  equals: cons!STATUS_IN_PROGRESS,
  then: "YELLOW",
  equals: cons!STATUS_CLOSED,
  then: "GREEN",
  default: "GRAY"
)
```

<a name="error-handling-patterns"></a>
### Error Handling Patterns

Handle errors and edge cases gracefully:

**Null-safe field access:**
```
a!defaultValue(
  ri!caseRecord[recordType!Case.fields.assignedTo],
  "Unassigned"
)
```

**Division by zero protection:**
```
a!localVariables(
  local!denominator: ri!totalCount,
  if(
    or(
      a!isNullOrEmpty(local!denominator),
      local!denominator = 0
    ),
    0,
    ri!successCount / local!denominator * 100
  )
)
```

**List bounds checking:**
```
a!localVariables(
  local!list: ri!items,
  local!index: ri!selectedIndex,
  if(
    and(
      a!isNotNullOrEmpty(local!list),
      local!index >= 1,
      local!index <= count(local!list)
    ),
    index(local!list, local!index, null),
    null
  )
)
```

**Cascading defaults:**
```
/* Use first non-null value */
if(
  a!isNotNullOrEmpty(ri!customName),
  ri!customName,
  if(
    a!isNotNullOrEmpty(ri!defaultName),
    ri!defaultName,
    "Unnamed"
  )
)
```

**Type-safe casting:**
```
a!localVariables(
  local!value: ri!inputValue,
  if(
    a!isNullOrEmpty(local!value),
    null,
    if(
      isnumber(local!value),
      tointeger(local!value),
      null
    )
  )
)
```

---

<a name="confirmation-patterns"></a>
## Confirmation Patterns

Expression rules require confirmation before destructive or breaking operations.

### Delete Confirmation

**When to use:** Before deleting an expression rule.

**Cross-reference:** Universal Workflow 1 (Delete Confirmation) in `references/confirmation-patterns.md`

**Object-specific checks:**
1. Check if rule is called by other rules (manual verification required)
2. Check if rule is used in interfaces (manual verification required)
3. Check if rule is used in process models (manual verification required)

**Template:**
```
⚠️ You are about to DELETE expression rule "CM_GetFullName"

⚠️ Cannot automatically detect all dependencies.

Manual verification REQUIRED:
1. Search expression rules for "rule!CM_GetFullName"
2. Search interfaces for "CM_GetFullName"
3. Search process models for script task expressions

Use Appian Designer:
- Right-click rule → "Find Usages" (if available)
- Review ALL calling locations

What breaks:
❌ All expression rules calling this rule will fail
❌ All interfaces using this rule will error
❌ All process models calling this rule will fail

Options:
1. Cancel (recommended until manual verification complete)
2. Proceed (you are responsible for updating ALL callers)

Type 'DELETE CM_GetFullName' to confirm.
```

### Name Collision Detection

**When to use:** Before creating a new expression rule.

**Cross-reference:** Universal Workflow 2 (Name Collision Detection) in `references/confirmation-patterns.md`

**Collision check:**
1. List existing expression rules in same application
2. Check for exact name match
3. Check for similar names (fuzzy match)

**Template:**
```
⚠️ Similar expression rule exists: "CM_GetUserName"

You want to create: "CM_GetFullName"

Similar existing rules:
- CM_GetUserName (returns username from User)
- CM_FormatName (formats name for display)

Options:
1. Create new "CM_GetFullName" anyway
2. Use existing "CM_GetUserName" instead
3. Use existing "CM_FormatName" instead

What would you like to do? (1/2/3)
```

<a name="breaking-change-detection"></a>
### Input Removal (Breaking Change)

**When to use:** Before removing an input from an expression rule.

**Risk level:** CRITICAL — breaks all callers using that input

**⚠️ IMPORTANT:** Use the [Expression Rule Breaking Changes Special Case](confirmation-patterns.md#expression-rule-breaking-changes-special-case) template from `references/confirmation-patterns.md`. This section shows the workflow; the Special Case provides the complete template structure.

**Workflow:**
1. Identify input to remove
2. Warn about breaking change
3. Present alternatives

**Template:**
```
⚠️ BREAKING CHANGE: Removing input "middleName" from CM_GetFullName

Current signature:
  CM_GetFullName(firstName, lastName, middleName)

Proposed signature:
  CM_GetFullName(firstName, lastName)

Impact:
❌ ALL callers passing middleName will fail
❌ Cannot automatically detect all callers (manual verification required)

Manual verification REQUIRED:
1. Search expression rules for "rule!CM_GetFullName(.*middleName"
2. Search interfaces for calls with 3 parameters
3. Update ALL callers to remove middleName parameter

Alternative approaches:
1. Make middleName optional instead (non-breaking)
2. Create CM_GetFullName_v2 with new signature
3. Cancel input removal

What would you like to do? (1/2/3)
```

---

<a name="common-pitfalls"></a>
## Pitfalls

### Syntax and Reference Errors

- **Using plain-text record type names** — `recordType!Board Committee Submission.fields.status` will fail because the parser can't resolve names with spaces (or any names) without UUIDs; always use the UUID-qualified format `'recordType!{uuid}Name.fields.{fieldUuid}fieldName'` (with single quotes wrapping the entire reference)

- **Forgetting `ri!` prefix** — Referencing inputs without `ri!` prefix treats them as undefined variables, causing runtime errors. `firstName` → error, `ri!firstName` → correct

- **Mismatched `ri!` names** — `ri!` names must exactly match the defined rule input or interface input name. Typo in input name or reference causes runtime error. Define: `firstName`, reference: `ri!fistName` → error

- **Missing `=` prefix in expression rule body** — Expression rules must start with `=`. Without it, the expression body is treated as literal text. Note: Within expressions (like `a!localVariables()`), the `=` is NOT used for assignments — `local!fullName: ri!firstName & " " & ri!lastName` is correct (no `=` prefix)

- **Using wrong reference prefix** — Each context has specific prefixes: `ri!` for rule inputs, `pv!` for process variables, `rv!` for record variables, `fv!` for function variables. Using `ri!` in process model → error, should be `pv!`

### Null Handling Errors

- **Using `=` for null comparison** — Use `a!isNullOrEmpty()` instead, since `null = null` returns `null`, not `true`. Correct: `a!isNullOrEmpty(ri!value)`, not `ri!value = null`. Use `a!isNotNullOrEmpty()` for the inverse check

- **Null propagation breaking calculations** — `null + 1` returns `null`, not `1`. Always guard with `a!defaultValue()` or `a!isNullOrEmpty()` check before operations

- **Forgetting null checks in `a!forEach()`** — If list can be null, `a!forEach(null, ...)` returns null. Wrap with `a!defaultValue(ri!list, {})` to ensure empty list

- **Index out of bounds on null list** — `index({}, 1, null)` returns null (safe), but `{}[1]` errors if list is null. Use `index()` function with default parameter

### Type Mismatch Errors

- **Dividing integers and expecting integer result** — `/` always returns Decimal, even for integer operands. `10 / 2` → `5.0` (Decimal), not `5` (Integer). Use `tointeger()` to convert if needed

- **Concatenating non-text without `&`** — Use `&` which auto-converts operands to text. `ri!count + " items"` → error if count is Integer. Correct: `ri!count & " items"` or `tostring(ri!count) & " items"`

- **Comparing incompatible types** — Comparing Decimal to Text auto-promotes to Text comparison. `1.5 = "1.5"` → `true` (both converted to Text). Be explicit with casting

- **Assigning wrong type to typed variable** — Process variables and rule inputs are typed. Assigning Text to Integer-typed variable causes error. Use explicit casting or fix type mismatch

### List Operation Errors

- **Assuming 0-based indexing** — Appian lists are 1-based. First item is `myList[1]`, not `myList[0]`. `myList[0]` → null

- **Forgetting `a!forEach()` returns a list** — Even for single item, result is `{value}` not `value`. Use `index(result, 1, null)` to extract single value from forEach result

- **Not flattening nested lists** — `a!forEach()` over list of lists creates nested structure `{{item1}, {item2}}`. Use `flatten()` to get `{item1, item2}`

- **Using `length()` instead of `count()`** — `count()` works on lists, `length()` is for strings. `length({1, 2, 3})` → error, use `count({1, 2, 3})`

### Performance Pitfalls

- **Nesting `if()` deeply** — More than 3-4 levels becomes unreadable and slow. Use `a!match()` for multi-branch logic:
  ```
  /* Avoid */
  if(x=1, "one", if(x=2, "two", if(x=3, "three", "other")))
  
  /* Better */
  a!match(
    value: x,
    equals: 1, then: "one",
    equals: 2, then: "two",
    equals: 3, then: "three",
    default: "other"
  )
  ```

- **Querying record types in loops** — Never call `a!queryRecordType()` inside `a!forEach()`. Query once, then transform results
  ```
  /* Avoid (N+1 queries) */
  a!forEach(items: ri!caseIds, expression: a!queryRecordType(recordType: recordType!Case, filters: a!queryFilter(field: recordType!Case.fields.id, operator: "=", value: fv!item)))
  
  /* Better (1 query) */
  a!localVariables(
    local!cases: a!queryRecordType(recordType: recordType!Case, filters: a!queryFilter(field: recordType!Case.fields.id, operator: "in", value: ri!caseIds)),
    local!cases.data
  )
  ```

- **Building large strings in loops** — String concatenation in loops is slow. Use `joinarray()` instead:
  ```
  /* Avoid */
  a!forEach(items: ri!names, expression: local!result & fv!item & ", ")
  
  /* Better */
  joinarray(ri!names, ", ")
  ```

- **Redundant `a!queryRecordType()` calls** — Cache query results in `local!` variables instead of re-querying:
  ```
  /* Avoid */
  if(a!queryRecordType(...).totalCount > 0, a!queryRecordType(...).data, {})
  
  /* Better */
  a!localVariables(
    local!result: a!queryRecordType(...),
    if(local!result.totalCount > 0, local!result.data, {})
  )
  ```

### Logic Errors

- **Circular references in expression rules** — Rule A calls Rule B, Rule B calls Rule A → runtime error. Design rules to avoid cycles

- **Not handling empty lists** — Many operations on empty lists return unexpected results. `sum({})` → null, not 0. Use `a!defaultValue(sum(ri!values), 0)`

- **Forgetting short-circuit evaluation** — `and()` and `or()` evaluate all arguments. Use `if()` for short-circuit:
  ```
  /* and() evaluates both (may error if myList is null) */
  and(a!isNotNullOrEmpty(ri!myList), count(ri!myList) > 5)
  
  /* if() short-circuits (safe) */
  if(a!isNotNullOrEmpty(ri!myList), count(ri!myList) > 5, false)
  ```

- **Incorrect operator precedence** — `and`, `or`, `not` are functions, not operators. Use parentheses for clarity:
  ```
  /* Unclear */
  and(x > 5, or(y = 1, y = 2))
  
  /* Clear */
  and(
    x > 5,
    or(y = 1, y = 2)
  )
  ```

---

<a name="best-practices"></a>
## Best Practices

### Reusability Patterns

**Extract reusable logic into expression rules:**
- Logic used in 2+ places → create expression rule
- Complex calculation (>3 lines) → create expression rule with descriptive name
- Business logic that might change → isolate in expression rule

**When to keep inline:**
- Simple field access: `rv!record[recordType!Case.fields.status]`
- One-off conditional: `if(ri!isUpdate, "Updated", "Created")`
- Visibility conditions: `a!isUserMemberOfGroup(loggedInUser(), cons!ADMIN_GROUP)`

**Rule composition over duplication:**
```
/* Build small, composable rules */
Rule: CM_GetFullName(firstName, lastName) → firstName & " " & lastName
Rule: CM_FormatUserDisplay(user) → rule!CM_GetFullName(firstName: user.firstName, lastName: user.lastName) & " (" & user.username & ")"

/* Not: CM_FormatUserDisplay duplicates name concatenation logic */
```

### Documentation Approaches

**Rule descriptions should state WHAT the rule returns:**
```
Good: "Returns full name by concatenating first and last name with a space."
Bad: "Gets the name" (unclear what format)
Bad: "Concatenates names" (unclear what it returns)
```

**Document input constraints in description:**
```
"Returns risk score (0-100) based on credit score. Returns 0 if creditScore is null or negative."
```

**Use comments for non-obvious logic:**
```
a!localVariables(
  /* Business rule: scores above 80 get 20% bonus, capped at 100 */
  local!bonusScore: if(ri!score > 80, ri!score * 1.2, ri!score),
  if(local!bonusScore > 100, 100, local!bonusScore)
)
```

**Document breaking changes in rule description:**
```
"[v2 - 2025-01-15] Added optional includeMiddle parameter. Previous callers continue working (non-breaking)."
```

### Testing Strategies

**Test expression rules independently before using in interfaces:**
1. Create test rule with sample inputs
2. Use `testRule` MCP tool to verify outputs
3. Test edge cases (nulls, empty lists, boundary values)
4. Only then use rule in interface or process model

**Test cases to always include:**
- Happy path (valid inputs, expected output)
- Null inputs (each input tested with null)
- Empty lists (if rule accepts lists)
- Boundary values (0, negative, very large numbers)
- Invalid combinations (if rule should validate)

**Use `a!localVariables()` for testable intermediate values:**
```
/* Testable: each local! can be validated */
a!localVariables(
  local!fullName: ri!firstName & " " & ri!lastName,
  local!isValid: a!isNotNullOrEmpty(local!fullName),
  local!formatted: if(local!isValid, upper(local!fullName), "UNKNOWN"),
  local!formatted
)

/* Hard to test: single complex expression */
if(a!isNotNullOrEmpty(ri!firstName & " " & ri!lastName), upper(ri!firstName & " " & ri!lastName), "UNKNOWN")
```

### Performance Considerations

**Query once, transform many:**
```
/* Efficient */
a!localVariables(
  local!cases: a!queryRecordType(recordType: recordType!Case, ...).data,
  local!openCases: where(a!forEach(items: local!cases, expression: fv!item.status = cons!STATUS_OPEN), local!cases),
  local!closedCases: where(a!forEach(items: local!cases, expression: fv!item.status = cons!STATUS_CLOSED), local!cases),
  ...
)

/* Inefficient (queries twice) */
a!localVariables(
  local!openCases: a!queryRecordType(filters: a!queryFilter(field: recordType!Case.fields.status, operator: "=", value: cons!STATUS_OPEN)),
  local!closedCases: a!queryRecordType(filters: a!queryFilter(field: recordType!Case.fields.status, operator: "=", value: cons!STATUS_CLOSED)),
  ...
)
```

**Cache expensive operations in local variables:**
```
a!localVariables(
  local!users: a!queryRecordType(recordType: recordType!User, ...).data,
  local!userCount: count(local!users),
  local!activeCount: count(where(a!forEach(items: local!users, expression: fv!item.isActive), local!users)),
  {
    totalUsers: local!userCount,
    activeUsers: local!activeCount,
    inactiveUsers: local!userCount - local!activeCount
  }
)
```

**Use appropriate batch sizes for queries:**
- Dropdowns/pickers: batchSize: 50-100
- Grids: batchSize: 20-50
- Aggregations only: batchSize: 1 (just get count)
- Full data export: batchSize: 500-1000 (paginated)

**Avoid nested `a!forEach()` when possible:**
```
/* Avoid O(n²) */
a!forEach(
  items: ri!list1,
  expression: a!forEach(
    items: ri!list2,
    expression: fv!item & " - " & fv!item
  )
)

/* Better: flatten first, then transform */
flatten(
  a!forEach(
    items: ri!list1,
    expression: ri!list2
  )
)
```

### Security Considerations

**Always validate user permissions before showing sensitive data:**
```
if(
  a!isUserMemberOfGroup(loggedInUser(), cons!ADMIN_GROUP),
  rv!record[recordType!Case.fields.sensitiveField],
  "[REDACTED]"
)
```

**Don't hard-code security checks — use constants:**
```
/* Good */
a!isUserMemberOfGroup(loggedInUser(), cons!CASE_MANAGERS_GROUP)

/* Bad - breaks when group renamed */
a!isUserMemberOfGroup(loggedInUser(), "Case Managers")
```

**Validate user input before using in queries:**
```
/* Prevent injection-style attacks */
a!localVariables(
  local!searchText: if(
    len(ri!searchText) > 100,
    left(ri!searchText, 100),
    ri!searchText
  ),
  a!queryRecordType(
    filters: a!queryFilter(
      field: recordType!Case.fields.title,
      operator: "includes",
      value: local!searchText
    )
  )
)
```

### Maintainability Guidelines

**Use meaningful variable names:**
```
/* Good */
local!filteredCases
local!totalAmount
local!isEligible

/* Bad */
local!temp
local!x
local!data
```

**Keep expressions readable — break into local variables:**
```
/* Readable */
a!localVariables(
  local!startDate: ri!caseRecord[recordType!Case.fields.createdAt],
  local!endDate: ri!caseRecord[recordType!Case.fields.closedAt],
  local!daysDiff: daysbetween(local!startDate, local!endDate),
  if(local!daysDiff > 30, "Overdue", "On Track")
)

/* Hard to read */
if(daysbetween(ri!caseRecord[recordType!Case.fields.createdAt], ri!caseRecord[recordType!Case.fields.closedAt]) > 30, "Overdue", "On Track")
```

**Consistent error handling across rules:**
```
/* Establish pattern and use everywhere */
a!map(
  success: local!isValid,
  data: if(local!isValid, local!result, null),
  error: if(local!isValid, null, local!errorMessage)
)
```

**Version control for breaking changes:**
- Rule name includes version: `CM_GetFullName_v2`
- Description documents version history
- Deprecate old version, migrate gradually

---

<a name="when-you-need-more"></a>
## When Documentation Lookup Is Needed

If expression validation fails with function-related errors and the function is not documented in function-reference.md, use the documentation search workflow defined in SKILL.md.

**Trigger conditions:**
1. `validateExpression` returns error mentioning unknown/invalid function
2. Function not found in function-reference.md
3. User explicitly asks about a function not in our docs

**Process:**
1. **Check if function exists** via functions.json (see SKILL.md for workflow)
2. If not found → function doesn't exist, check anti-hallucination list for alternatives
3. If found → fetch documentation page
4. **Extract signature** and create expression using official documentation
5. **Re-validate** with `validateExpression` tool
6. **Optional:** If function is commonly used, suggest adding to function-reference.md

**Version:** Uses the configured version from SKILL.md Configuration section.

**Automatic vs Manual:** Automatically fetch docs when function not found. No user prompt needed - fetching is fast and non-disruptive.

---

## When You Need More

This file covers expression patterns and best practices. For object-specific guidance:
- Record type queries: `references/record-types.md`
- Security expressions: `references/security-patterns.md`
- Process model expressions: `references/process-models.md`
- Interface expressions: `references/interfaces.md`
