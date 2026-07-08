# SAIL Validation Checkpoint

**Purpose:** Validate SAIL expressions BEFORE creating objects in Appian to catch errors early and enable automatic retry.

**When to use:** After generating SAIL code for interfaces or expression rules, before calling `createInterface` or `createExpressionRule`.

**Pattern origin:** Adopted from composer-lightweight's post-generation validation pattern (Pattern 2).

---

## Why This Matters

**Without pre-validation:**
- ❌ Errors discovered AFTER object creation fails in Appian
- ❌ Manual fix required → manual update call
- ❌ Interface/rule created with errors (if validation is weak)
- ❌ No structured retry mechanism

**With pre-validation:**
- ✅ Errors caught BEFORE attempting to create object
- ✅ Automatic retry loop (up to 2 attempts)
- ✅ Only creates object after validation passes
- ✅ Cleaner separation: Generate → Validate → Fix → Create

---

## Important Limitations: ri! References in Validation

### Expression Rules with ri! References

**⚠️ `validateExpression` cannot validate expression rule bodies that use `ri!` references.**

The tool validates standalone SAIL expressions without rule input context, so expressions like `ri!quantity * ri!unitPrice` will fail validation with "Unresolved reference(s): ri!quantity, ri!unitPrice" even if the inputs are correctly defined.

**Workaround for expression rules:**
1. **Validate wrapped version** - Wrap the expression in `a!localVariables()` with test values:
   ```sail
   a!localVariables(
     local!quantity: 10,
     local!unitPrice: 5.99,
     /* Expression body using local! instead of ri! */
     local!quantity * local!unitPrice
   )
   ```
2. **If validation passes** → Replace `local!` with `ri!` in actual expression rule
3. **Alternative:** Skip Step 7B for simple expression rules with only `ri!` references and arithmetic/functions

### Interfaces with ri! References

**✅ Interfaces CAN use `ri!` references successfully, but validation has limitations.**

Interfaces that accept rule inputs (e.g., `ri!caseId`, `ri!orderId`) will fail `validateExpression` with "Unresolved reference" errors because the validation tool doesn't have the input context.

**Validation approach for interfaces with ri!:**

**Option A (Recommended):** Skip `validateExpression` and proceed directly to `createInterface` with inputs defined
```
1. Generate SAIL expression with ri! references
2. Skip Step 7B validation
3. Call createInterface with inputs parameter:
   {
     name: "CM_CaseDetails",
     expression: "<SAIL with ri!caseId>",
     inputs: [
       {name: "caseId", type: "Number (Integer)"}
     ]
   }
4. Appian validates the expression when the interface is created
```

**Option B:** Validate a modified version first (for complex interfaces):
```
1. Replace ri! with local! + test values
2. Validate the local! version
3. If passes → Replace local! back to ri!
4. Call createInterface with ri! version + inputs
```

**When to skip Step 7B validation for interfaces:**
- ✅ Interface uses `ri!` references for rule inputs
- ✅ Interface is straightforward with standard components
- ✅ All functions and components were verified in Step 4A/4B
- ⚠️ Complex interface with heavy logic → Consider validating simplified version first

**Validation DOES work for:**
- ✅ Interfaces using only `local!` variables (no rule inputs)
- ✅ Expression rules using only functions and literals (no `ri!` references)
- ✅ Expression rules wrapped in `a!localVariables()` with `local!` instead of `ri!`

**Recommendation:**
- **Interfaces with ri! inputs:** Skip Step 7B validation, proceed to createInterface
- **Interfaces without ri!:** Always use Step 7B validation (MANDATORY)
- **Expression rules (simple):** Validate wrapped version or skip if only basic arithmetic with `ri!`
- **Expression rules (complex):** Always validate wrapped version to catch function/logic errors

---

## Validation Workflow

### Step 1: Generate SAIL Code

Generate the SAIL expression using all loaded references and patterns (Steps 1-6 complete).

**For Expression Rules:**
- Create the expression body as a string
- Ensure all `ri!` references match input parameter names
- Apply null safety patterns
- Use verified functions only (Step 4A complete)

**For Interfaces:**
- Create the full interface expression as a string
- Ensure all `ri!` references match input parameter names
- Apply null safety patterns
- Use verified functions AND components (Step 4A + 4B complete)
- Follow interface-generation-checklist.md

### Step 2: Validate Expression (BEFORE Creating Object)

**Call `validateExpression` MCP tool:**

```json
{
  "expression": "<full SAIL expression string>",
  "bindings": null
}
```

**Expected responses:**

✅ **Success** (no errors):
```json
{
  "hasErrors": false,
  "parseErrors": [],
  "discoveryErrors": [],
  "evalErrors": []
}
```

❌ **Failure** (has errors):
```json
{
  "hasErrors": true,
  "parseErrors": ["Line 5: unexpected token ')'"],
  "discoveryErrors": [],
  "evalErrors": []
}
```

### Step 3: Handle Validation Results

#### If Validation Passes (hasErrors = false):
✅ Proceed to Step 4 (Create Object)

#### If Validation Fails (hasErrors = true):
❌ Enter retry loop

**Retry Loop (MAX_ATTEMPTS = 3):**

```
Attempt 1: Generate SAIL
  → Validate
  → If fails: Fix errors based on validation response

Attempt 2: Re-validate fixed SAIL
  → If fails: Fix errors again

Attempt 3: Final validation
  → If fails: Report to user with error details
  → If passes: Proceed to Step 4
```

**Error Fix Strategy:**

1. **Parse errors** (syntax issues):
   - Missing parentheses, brackets, or quotes
   - Invalid function names (check anti-hallucination list)
   - Malformed parameters

2. **Discovery errors** (reference issues):
   - Unknown `ri!` variable names
   - Invalid `recordType!` or `local!` references
   - Missing constants or record types

3. **Eval errors** (type/logic issues):
   - Type mismatches (Text vs Number)
   - Null safety violations
   - Invalid operators

**For each attempt:**
- Extract error message from validation response
- Identify root cause (syntax, reference, type)
- Apply fix to SAIL expression
- Re-validate

**After 3 failed attempts:**
- Report to user: "Validation failed after 3 attempts"
- Include final error messages
- Ask user for guidance or manual review

### Step 4: Create Object (Only After Validation Passes)

✅ **Validation passed** → Now safe to call:

**For Expression Rules:**
```
Call: createExpressionRule
Parameters: {
  name: "...",
  expression: "<validated SAIL>",
  inputs: [...],
  ...
}
```

**For Interfaces:**
```
Call: createInterface
Parameters: {
  name: "...",
  expression: "<validated SAIL>",
  inputs: [...],
  ...
}
```

---

## Code Pattern (Pseudo-code)

```javascript
// Step 1: Generate SAIL
const sailExpression = generateSAIL(requirements)

// Step 2-3: Validate with retry loop
const MAX_ATTEMPTS = 3
let validationPassed = false
let currentExpression = sailExpression

for (let attempt = 1; attempt <= MAX_ATTEMPTS; attempt++) {
  console.log(`Validation attempt ${attempt}/${MAX_ATTEMPTS}`)
  
  const validation = await validateExpression({
    expression: currentExpression,
    bindings: null
  })
  
  if (!validation.hasErrors) {
    console.log("✅ Validation passed")
    validationPassed = true
    break
  }
  
  // Validation failed
  console.log(`❌ Validation failed: ${extractErrors(validation)}`)
  
  if (attempt < MAX_ATTEMPTS) {
    // Fix and retry
    currentExpression = fixErrors(currentExpression, validation)
  }
}

// Step 4: Create object (only if validation passed)
if (validationPassed) {
  if (isExpressionRule) {
    await createExpressionRule({...})
  } else {
    await createInterface({...})
  }
} else {
  throw new Error("Validation failed after 3 attempts. Manual review required.")
}
```

---

## Example: Expression Rule Validation

**Scenario:** Create expression rule to calculate order total

### Attempt 1: Generate and Validate

```sail
ri!quantity * ri!unitPrice
```

**Validation call:**
```json
{
  "expression": "ri!quantity * ri!unitPrice"
}
```

**Validation response:**
```json
{
  "hasErrors": false
}
```

✅ **Pass** → Proceed to create expression rule

---

## Example: Interface Validation with Retry

**Scenario:** Create interface with KPI cards

### Attempt 1: Generate and Validate

```sail
a!headerContentLayout(
  header: {
    a!columnsLayout(
      columns: {
        a!columnLayout(
          contents: {
            a!cardLayout(
              contents: {
                a!richTextDisplayField(
                  value: {
                    a!richTextItem(text: "Total Revenue", size: "MEDIUM")
                  }
                )
              }
            )
          }
        )
      }
    )
  }
)
```

**Validation call:**
```json
{
  "expression": "<above SAIL>"
}
```

**Validation response:**
```json
{
  "hasErrors": true,
  "parseErrors": ["Line 2: 'header' parameter expects a list of components, received single component"]
}
```

❌ **Fail** → Fix error

### Attempt 2: Fix and Re-validate

**Fix:** Wrap header value in array `{...}`

```sail
a!headerContentLayout(
  header: a!columnsLayout(
    columns: {
      a!columnLayout(
        contents: {
          a!cardLayout(
            contents: {
              a!richTextDisplayField(
                value: {
                  a!richTextItem(text: "Total Revenue", size: "MEDIUM")
                }
              )
            }
          )
        }
      )
    }
  )
)
```

**Validation response:**
```json
{
  "hasErrors": false
}
```

✅ **Pass** → Proceed to create interface

---

## Integration with Existing Workflow

This checkpoint is **Step 7B** in the overall workflow:

1. **Step 1-6:** Load references, verify functions/components, complete pre-implementation checks
2. **Step 7A:** Generate SAIL expression
3. **Step 7B:** Validate expression with retry loop ← **THIS CHECKPOINT**
4. **Step 7C:** Create object in Appian (only after validation passes)

---

## Key Benefits

1. **Catch errors early** — Before attempting to create object
2. **Automatic retry** — Up to 3 attempts to fix validation errors
3. **Structured workflow** — Clear separation: Generate → Validate → Fix → Create
4. **Better user experience** — Fewer failed object creations, cleaner error messages
5. **Consistency** — Same validation approach for both expression rules and interfaces

---

## Common Validation Errors and Fixes

| Error Type | Common Cause | Fix |
|------------|-------------|-----|
| Parse error: "unexpected token" | Missing/extra parentheses, brackets | Balance delimiters |
| Discovery error: "unknown variable ri!xyz" | Typo in rule input name | Match input parameter name exactly |
| Discovery error: "unknown function abc()" | Function doesn't exist | Check anti-hallucination list, use alternative |
| Eval error: "type mismatch" | Wrong type (Text vs Number) | Apply explicit casting (tointeger, totext) |
| Parse error: "parameter 'x' expects Y" | Wrong parameter type | Check component-reference.md for correct type |

---

## Notes

- This pattern is inspired by composer-lightweight's `generate_interfaces/handler.py` lines 169-201
- `validateExpression` is an Appian MCP tool (not a custom script)
- Retry loop is client-side logic (not built into the tool)
- Validation is **syntax and structure only** — it doesn't verify business logic correctness
- Use `testRule` or `testInterface` for runtime/integration testing (separate step after creation)
