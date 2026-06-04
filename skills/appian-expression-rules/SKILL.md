---
name: "appian-expression-rules"
description: "Create and modify Appian expression rules including rule inputs, expression bodies, and naming conventions. Covers when to use expression rules vs. inline expressions, rule input definition with QName types, and the ri! input reference pattern. Use when creating, updating, or managing expression rules."
---

## Relevant Tools

Expression rule tools cover the full lifecycle:

- **Create an expression rule** — provide a name, and optionally a description, expression body, and rule inputs
- **List and get expression rules** — retrieve expression rules by UUID or search with optional filtering; scope to an application with `appUuid`
- **Update an expression rule** — modify name, description, expression body, or rule inputs on an existing expression rule
- **Delete an expression rule** — permanently remove an expression rule by UUID

## When to Create an Expression Rule

Create an expression rule when:

- Logic is reused across multiple objects (interfaces, process models, other expression rules)
- The logic is complex enough to benefit from a descriptive name
- The logic needs independent testing or documentation
- You want to encapsulate a calculation, transformation, or query that other objects call

Use inline expressions instead when:

- The logic is simple and used in only one place (e.g., a single XOR condition, a one-off visibility check, a script task assignment)
- Creating a named rule would add indirection without adding clarity

For expression syntax, operators, type system, and common patterns, activate the expressions knowledge skill. This skill focuses on the expression rule object lifecycle and conventions.

## Creating Expression Rules

### Naming Conventions

- **Prefix**: Use the application prefix followed by an underscore (e.g., `ACME_`)
- **Pattern**: `[PREFIX]_[DescriptiveName]` in PascalCase
- **Name should describe what the rule returns or does**, not how it works
- **Examples**:
  - `ACME_GetFullName` — concatenates first and last name
  - `ACME_IsEligibleForEscalation` — returns Boolean based on case status and age
  - `ACME_FormatCaseReference` — builds a formatted reference string from case fields
  - `ACME_CalculateRiskScore` — computes a risk score from input parameters
  - `ACME_GetOpenCaseCount` — returns a count of open cases for a user

### Rule Inputs

Rule inputs are the parameters that callers pass into the expression rule. Each input needs:

- `name` — camelCase identifier (e.g., `firstName`, `caseRecord`, `searchText`). This becomes the `ri!` reference inside the expression body.
- `type` — Appian type in QName format (see type table below)
- `description` — optional but recommended; explains what the input represents

### Input Type QNames

When defining rule inputs, use the Appian type QName format:

| Friendly Type | QName |
|---|---|
| Text | `{http://www.appian.com/ae/types/2009}Text` |
| Integer | `{http://www.appian.com/ae/types/2009}Integer` |
| Decimal | `{http://www.appian.com/ae/types/2009}Decimal` |
| Boolean | `{http://www.appian.com/ae/types/2009}Boolean` |
| Date | `{http://www.appian.com/ae/types/2009}Date` |
| DateTime | `{http://www.appian.com/ae/types/2009}DateTime` |
| Time | `{http://www.appian.com/ae/types/2009}Time` |
| User | `{http://www.appian.com/ae/types/2009}User` |
| Group | `{http://www.appian.com/ae/types/2009}Group` |

For record type inputs, use the record type's own type QName (retrieved from the record type object).

### Expression Body

The expression body is the logic the rule evaluates. Key rules:

- The expression must start with `=`
- Reference inputs using the `ri!` prefix matching the input names you defined (e.g., input named `firstName` → `ri!firstName`)
- The expression can call other expression rules via `rule!PREFIX_RuleName(inputName: value)`
- The expression can reference constants via `cons!CONSTANT_NAME`, record types via `recordType!EntityName`, and other Appian objects using standard reference prefixes

### Operation Sequence

When creating an expression rule with inputs and an expression body, you can provide everything in a single create call:

1. Define the `name` and optional `description`
2. Define `inputs` — the list of rule input definitions (name, type, description)
3. Define `expression` — the expression body that references those inputs via `ri!`

Alternatively, create the rule first and update it later to add inputs and the expression body.

When updating an existing expression rule:

1. Get the current expression rule to see its existing inputs and expression
2. Modify the inputs or expression as needed
3. If changing inputs, ensure the expression body still references the correct `ri!` names

## Calling Expression Rules

Other Appian objects call expression rules using the `rule!` prefix with named parameters:

```
rule!ACME_GetFullName(firstName: "Jane", lastName: "Doe")
rule!ACME_IsEligibleForEscalation(caseRecord: rv!record)
rule!ACME_CalculateRiskScore(score: ri!creditScore)
```

Expression rules can be called from:
- Interface SAIL expressions
- Other expression rules
- Process model script tasks
- XOR gateway conditions (if the rule returns Boolean)

## Conventions

### Descriptions

Write descriptions that explain what the rule does and what it returns:
- "Returns the full name by concatenating first and last name with a space."
- "Returns true if the case is eligible for escalation based on status and age."
- "Formats a case reference string from the case ID and creation date."

### Keep Rules Focused

Each expression rule should do one thing well. If a rule is growing complex with multiple responsibilities, split it into smaller rules that compose together.

### Input Naming

- Use camelCase for input names: `firstName`, `caseRecord`, `searchText`
- Name inputs after what they represent, not their type: `caseRecord` not `recordInput`
- For Boolean inputs, use question-style names: `isUpdate`, `includeArchived`

## Common Pitfalls

- **Using plain-text record type names in expression bodies** — all `recordType!` references in expressions submitted via the API must use the UUID-qualified format wrapped in single quotes: `'recordType!{rtUuid}Name.fields.{fieldUuid}fieldName'`. See the Appian Expressions skill for the full format specification.
- **Mismatched `ri!` names** — the `ri!` references in the expression body must exactly match the input names defined on the rule; a typo causes a runtime error, not a compile error
- **Forgetting the `=` prefix** — expression bodies must start with `=`; omitting it causes the entire expression to be treated as a literal text string
- **Wrong QName format for input types** — the namespace must be exactly `{http://www.appian.com/ae/types/2009}`; a typo in the namespace silently fails
- **Updating inputs without updating the expression** — if you rename or remove an input, the expression body still references the old `ri!` name; always update both together
- **Creating rules for one-off logic** — if the logic is only used in one place and is simple, an inline expression is cleaner; don't create a rule just because you can
- **Circular references** — expression rules can call other expression rules, but circular dependencies (A calls B, B calls A) cause runtime errors
- **Not using the application prefix** — expression rules without a prefix are hard to find and may collide with rules from other applications in the same environment

## When You Need More

For expression syntax, operators, type system, object reference patterns, and common functions, activate the expressions knowledge skill. This skill focuses on the expression rule object lifecycle; the expressions skill covers how to write the code inside them.

For questions about specific Appian expression rule behavior or advanced patterns beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.
