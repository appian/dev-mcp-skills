# SAIL Verification Checkpoint

🛑 **MANDATORY: You cannot proceed to code generation without completing this checkpoint.**

This checkpoint prevents hallucinating non-existent functions or SAIL components. Complete the relevant section(s) based on what you're building:

- **Expression rules:** Complete **Step 3A only** (verify functions)
- **Interfaces:** Complete **BOTH Step 3A and Step 3B** (verify functions AND components)

**Why both for interfaces?** Every interface uses:
- **Components** for UX (a!cardLayout, a!textField, a!gridField)
- **Functions** for logic (a!localVariables, a!recordData, if(), a!queryFilter, etc.)

---

## Step 3A: Verify Functions [MANDATORY EXECUTION]

**Required for:** Expression rules AND interfaces

🛑 **YOU MUST EXECUTE THESE COMMANDS AND OUTPUT THE RESULTS.**

### 1. List All Functions You Plan to Use

Write them down. Example:
```
Functions I will use:
- a!localVariables
- a!recordData
- a!queryFilter
- a!map
- if
- a!isNullOrEmpty
- tostring
- index
```

### 2. Check Anti-Hallucination List [EXECUTE NOW]

**Command to run:**
```bash
grep -A 20 "Functions That DO NOT Exist" references/function-reference.md | head -30
```

**You MUST:**
1. Run this command
2. Read the output
3. Check your function list against it
4. Remove any functions on the NO-EXIST list

**Common non-existent functions to avoid:**
- ❌ `regexmatch()` → Use `like()` or `search()` instead
- ❌ `property()` → Use `index()` or dot notation instead
- ❌ `a!recordQuery()` → Use `a!recordData()` instead
- ❌ `a!dateTimeValue()` → Use `datetime()` instead

### 3. Verify Functions via Tier 2A (Optional - for unknown functions)

**Only if you have functions NOT in function-reference.md:**

```bash
VERSION="26.6"
# For each unknown function:
curl -sL "https://docs.appian.com/suite/help/$VERSION/functions.json" | jq -r '.["functionname"]'
```

If curl fails: Skip this step and proceed with Tier 1 only.

### 4. Output Required Verification Artifacts [MANDATORY]

🚫 **YOU CANNOT PROCEED WITHOUT OUTPUTTING THIS:**

```
=== STEP 4A VERIFICATION COMPLETE ===
Functions to verify: [paste your actual function list here]
Anti-hallucination check: ✅ (ran grep command, checked lines [X-Y])
Functions on NO-EXIST list found: [count or "none" - list them if >0]
Tier 2A verification: [✅ verified N functions | ⏭️ skipped - all in reference]
Functions verified as existing: [count]/[total]
Non-existent functions removed: [count or "none" - list them if >0]
Ready to proceed: [YES or NO]
```

**Example filled template:**
```
=== STEP 4A VERIFICATION COMPLETE ===
Functions to verify: a!localVariables, a!queryRecordByIdentifier, a!defaultValue, a!writeRecords, a!save, if, user, text, trim
Anti-hallucination check: ✅ (ran grep command, checked lines 27-40)
Functions on NO-EXIST list found: none
Tier 2A verification: ⏭️ skipped - all in reference
Functions verified as existing: 9/9
Non-existent functions removed: none
Ready to proceed: YES
```

🚫 **BLOCKING GATE:** You MUST output the filled template above before proceeding to Step 3B or Step 5.

✅ **Step 3A complete. Expression rules: proceed to Step 4. Interfaces: continue to Step 3B.**

---

## Step 3B: Verify SAIL Components [MANDATORY EXECUTION]

**Required for:** Interfaces only (skip this for expression rules)

🛑 **YOU MUST EXECUTE THESE COMMANDS AND OUTPUT THE RESULTS.**

### 1. List All SAIL Components You Plan to Use

Write them down. Example:
```
Components I will use:
- a!formLayout
- a!textField
- a!dropdownField
- a!buttonWidget
- a!pickerFieldUsers
```

**Note:** Some Appian constructs are functions, not components (e.g., `a!localVariables`, `a!save`, `a!map`). If unsure, check both Step 3A and Step 3B.

### 2. Check Component Existence via Registry [EXECUTE NOW]

**Command to run:**
```bash
# Check all your components at once:
grep -E '"a!(formLayout|textField|dropdownField|buttonWidget|pickerFieldUsers)"' registry/components-registry.json | head -20
```

**You MUST:**
1. Run this command (replace component names with your actual list)
2. Verify each component has `"exists": true`
3. Remove any with `"exists": false` from your plan

### 3. Load Instruction Files [EXECUTE NOW]

**Command to run:**
```bash
# Check which of your components have instruction files:
jq -r '.["a!formLayout"].instructionFile // "none", .["a!textField"].instructionFile // "none", .["a!dropdownField"].instructionFile // "none", .["a!pickerFieldUsers"].instructionFile // "none"' registry/components-registry.json
```

**For each component with an instruction file path:**
- Load the file: `Read [path]` (relative to the skill root)
- Note any critical warnings

**Common critical warnings to watch for:**
- `a!formLayout` → Cannot be nested
- `a!pickerFieldUsers` → maxSelections defaults to unlimited
- `a!dropdownField` → Query lookup tables, don't hardcode

### 4. Output Required Verification Artifacts [MANDATORY]

🚫 **YOU CANNOT PROCEED WITHOUT OUTPUTTING THIS:**

```
=== STEP 4B VERIFICATION COMPLETE ===
Components to verify: [paste your actual component list here]
Registry check: ✅ (ran grep command, all exist=true)
Components with instruction files:
  - [component] → [file path] ✅ loaded
  - [component] → [file path] ✅ loaded
  - [component] → none (no instruction file)
Critical warnings noted:
  1. "[direct quote from instruction file]"
  2. "[direct quote from instruction file]"
Tier 2B verification: [✅ N components | ⏭️ skipped - all have instruction files]
Non-existent components removed: [count or "none" - list them if >0]
Ready to proceed: [YES or NO]
```

**Example filled template:**
```
=== STEP 4B VERIFICATION COMPLETE ===
Components to verify: a!formLayout, a!sectionLayout, a!textField, a!dropdownField, a!pickerFieldUsers, a!buttonWidget, a!buttonLayout
Registry check: ✅ (ran grep command, all exist=true)
Components with instruction files:
  - a!formLayout → layouts/form-layout-instructions.md ✅ loaded
  - a!pickerFieldUsers → components/picker-field-users-instructions.md ✅ loaded
  - a!textField → none
  - a!dropdownField → none
  - a!buttonWidget → none
Critical warnings noted:
  1. "a!formLayout cannot be nested inside another a!formLayout"
  2. "a!pickerFieldUsers: maxSelections defaults to unlimited, set to 1 for single user"
Tier 2B verification: ⏭️ skipped - all components have instruction files or are in component-reference.md
Non-existent components removed: none
Ready to proceed: YES
```

🚫 **BLOCKING GATE:** You MUST output the filled template above before proceeding to Step 3C.

✅ **Step 3B complete. Interfaces: continue to Step 3C (parameter verification).**

---

## Step 3C: Verify Component Parameters [MANDATORY EXECUTION]

**Required for:** Interfaces only (skip this for expression rules)

🛑 **YOU MUST EXECUTE THESE COMMANDS AND LIST PARAMETERS.**

### 1. List All Parameters You Plan to Use For Each Component

For EACH component, write down the parameters. Example:
```
a!formLayout:
  - titleBar (for title)
  - contents
  - buttons

a!sectionLayout:
  - label
  - contents
  - divider (NOT showBorder - doesn't exist)

a!dropdownField:
  - label
  - choiceLabels (REQUIRED even when using data param)
  - choiceValues (REQUIRED even when using data param)
  - data (optional record-type source; does NOT replace choiceLabels/choiceValues)
  - value
  - saveInto
  - disabled (NOT readOnly - doesn't exist)
```

### 2. Check Critical Parameter Warnings [EXECUTE NOW]

**Command to run:**
```bash
# Check for critical warnings on your components:
jq -r 'to_entries[] | select(.value.criticalWarnings != null) | "\(.key): \(.value.criticalWarnings | join("; "))"' registry/components-registry.json | grep -E "(formLayout|sectionLayout|dropdownField|checkboxField)"
```

**You MUST:**
1. Run this command
2. Read ALL critical warnings for your components
3. Verify your parameter list doesn't violate any warnings

**Common violations to avoid:**
- ❌ `a!formLayout` with `label` → Use `titleBar` instead
- ❌ `a!sectionLayout` with `showBorder` → Use `divider` or wrap in `a!cardLayout`
- ❌ `a!dropdownField` with `readOnly` → Use `disabled` instead
- ❌ `a!checkboxField` with `readOnly` → Use `disabled` instead

### 3. Cross-Reference Parameters Against Instruction Files

**For each component WITH an instruction file:**

1. Load the instruction file (you did this in Step 3B)
2. Find the "Parameters" or "Common Parameters" table
3. **Verify EVERY parameter you listed exists in that table**

**For each component WITHOUT an instruction file:**

1. Load component-reference.md
2. Find the component's parameter table
3. **Verify EVERY parameter you listed exists in that table**

**Command to quickly check component-reference.md:**
```bash
# Example: Check if titleBar exists for a!formLayout
grep -A 20 "^### a!formLayout$" references/component-reference.md | grep -E "^\| \`titleBar\`"

# Example: Check if label exists for a!formLayout (should return nothing)
grep -A 20 "^### a!formLayout$" references/component-reference.md | grep -E "^\| \`label\`"
```

### 4. Output Required Verification Artifacts [MANDATORY]

🚫 **YOU CANNOT PROCEED WITHOUT OUTPUTTING THIS:**

```
=== STEP 3C PARAMETER VERIFICATION COMPLETE ===
Components with parameters to verify: [count]

For each component:
  [ComponentName]:
    Parameters: [param1, param2, param3]
    Critical warnings checked: [✅ no violations | ⚠️ violations found and fixed]
    Parameter existence verified: [✅ all exist | ❌ N parameters don't exist]
    Non-existent parameters removed: [list them or "none"]

Critical warnings reviewed: [count] warnings
Parameters verified against instruction files: [count]/[total]
Parameters verified against component-reference.md: [count]/[total]
Non-existent parameters found: [count or "none" - list them if >0]
Parameter violations fixed: [count or "none" - list them if >0]

Ready to proceed: [YES or NO]
```

**Example filled template:**
```
=== STEP 3C PARAMETER VERIFICATION COMPLETE ===
Components with parameters to verify: 5

For each component:
  a!formLayout:
    Parameters: titleBar, contents, buttons
    Critical warnings checked: ✅ no violations (using titleBar not label)
    Parameter existence verified: ✅ all exist
    Non-existent parameters removed: none
    
  a!sectionLayout:
    Parameters: label, contents, divider
    Critical warnings checked: ✅ no violations (using divider not showBorder)
    Parameter existence verified: ✅ all exist
    Non-existent parameters removed: showBorder (replaced with divider)
    
  a!dropdownField:
    Parameters: label, choiceLabels, choiceValues, value, saveInto, disabled
    Critical warnings checked: ✅ no violations (using disabled not readOnly)
    Parameter existence verified: ✅ all exist
    Non-existent parameters removed: readOnly (replaced with disabled)
    
  a!textField:
    Parameters: label, value, readOnly
    Critical warnings checked: ✅ no violations (textField supports readOnly)
    Parameter existence verified: ✅ all exist
    Non-existent parameters removed: none
    
  a!buttonWidget:
    Parameters: label, value, saveInto
    Critical warnings checked: ✅ no violations
    Parameter existence verified: ✅ all exist
    Non-existent parameters removed: none

Critical warnings reviewed: 4 warnings (formLayout, sectionLayout, dropdownField, checkboxField)
Parameters verified against instruction files: 3/15 (formLayout)
Parameters verified against component-reference.md: 12/15 (others)
Non-existent parameters found: 2 (showBorder, readOnly for dropdown)
Parameter violations fixed: 2 (showBorder → divider, readOnly → disabled)

Ready to proceed: YES
```

🚫 **BLOCKING GATE:** You MUST output the filled template above before proceeding to Step 4.

✅ **Step 3C complete. Return to SKILL.md to continue with Step 4 (supplementary references) and Step 5 (pre-implementation verification).**

---

## Final Verification Summary

Before generating code, complete this summary:

### Expression Rules
- [ ] Step 3A completed (all functions verified)
- [ ] No non-existent functions in code plan
- [ ] Ready to proceed to Step 4

### Interfaces
- [ ] Step 3A completed (all functions verified)
- [ ] Step 3B completed (all components verified)
- [ ] Step 3C completed (all component parameters verified)
- [ ] Instruction files loaded and critical warnings noted
- [ ] No non-existent functions, components, or parameters in code plan
- [ ] Ready to proceed to Step 4

---

## Quick Reference

**Related documentation:**
- Anti-hallucination lists: `function-reference.md`, `component-reference.md`
- Function catalog: `function-reference.md` (curated list of ~50 common functions)
- Component catalog: `component-reference.md` (~53 common components)
- Component registry: `registry/components-registry.json` (all 145 components)
- Full lookup workflow: `documentation-lookup-strategy.md`
- Component instruction file index: `component-loading-index.md`

**Verification tier summary:**
- **Tier 1:** Skill references (anti-hallucination lists, curated catalogs)
- **Tier 2A:** functions.json for function existence (495 functions, definitive)
- **Tier 2B:** Registry + functions.json for component existence (145 components)
- **Tier 3:** Documentation search for patterns/recipes (optional, when Tier 1/2 insufficient)

**Path to files** (all relative to the skill root):
- References: `references/`
- Registry: `registry/components-registry.json`
- Component instructions: `components/` and `layouts/`

---

🎯 **Checkpoint complete. You are now ready to generate code with verified functions and components.**
