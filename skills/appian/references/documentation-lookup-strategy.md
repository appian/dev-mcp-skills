# Documentation Lookup Strategy

**Context:** This reference details the three-tier documentation lookup strategy mentioned in SKILL.md Step 3. Use this when you need to look up functions, patterns, or recipes beyond the loaded skill references.

**When to load this file:**
- Step 3 checkpoint redirects you here for tier details
- You need curl commands for functions.json lookups
- You need error handling guidance for tier fallbacks
- You need to understand tier selection logic

---

## Overview: Three-Tier Fallback Chain

When you need information beyond the loaded skill references, use this three-tier approach. Each tier has specific strengths - use the right tool for the task.

**Fallback Chain (query-type specific):**

**For function existence/signature queries:**
1. **Tier 1** — Check anti-hallucination list in function-reference.md
   - If listed as non-existent → **STOP** (definitive "No", provide alternatives)
2. **Tier 1** — Check function-reference.md for documented functions
   - If fully documented → Use Tier 1 (no further lookup needed)
   - If partially documented or not found → Proceed to Tier 2
3. **Tier 2** — Check functions.json (definitive yes/no for existence, complete signature)
4. **If Tier 2 fails** → Skip Tier 3 (unreliable for existence checks), go to best-effort
5. **Best-effort** — Use Tier 1 only, inform user function existence could not be verified

**For patterns/recipes/best practices queries:**
1. **Tier 1** — Check skill references first
   - If fully documented → Use Tier 1 (no further lookup needed)
   - If insufficient → Proceed to Tier 3
2. **Tier 3** — Search official documentation (skip Tier 2, not applicable)
3. **If Tier 3 fails** → Best-effort with Tier 1 only
4. **Best-effort** — Use Tier 1 only, inform user official patterns unavailable

**For mixed queries (function + pattern):**
1. **Tier 1** — Check skill references for both aspects
2. **Tier 2** — Look up function existence/signature (if needed)
3. **Tier 3** — Search for patterns/recipes (if needed)
4. **Combine results** from both Tier 2 and Tier 3 with Tier 1 context

**Rationale:** Tier 3 cannot definitively confirm function non-existence (may return related results). For function lookups, failed Tier 2 means no definitive answer is possible—skip Tier 3 to avoid false positives. Anti-hallucination list in Tier 1 is authoritative for known non-existent functions.

---

## Tier 1: Skill References (Always Check First)

**What it provides:**
- Curated patterns and examples
- Anti-patterns and gotchas
- May exceed official docs in detail
- Custom examples for complex scenarios

**When it's sufficient:**
- Function is well-documented in function-reference.md
- Pattern files exist (array-patterns.md, match-foreach-patterns.md, etc.)
- Detailed examples and usage notes are present

**Example:** `a!match()` is fully documented in match-foreach-patterns.md with complete structure, examples, and rules - no further lookup needed.

**If skill references lack detail, proceed to Tier 2 or 3.**

---

## Tier 2: functions.json + curl (Precise Function Lookups)

**What it provides:**
- Definitive existence check (function exists or doesn't)
- Official function signatures
- Fast, deterministic lookups
- Version-specific precision

**Use for:**
- ✅ **Existence checks** - "Does `regexmatch()` exist?" (definitive yes/no)
- ✅ **Function signature verification** - Parameters, types, required/optional
- ✅ **Unknown functions** - Expression validation error mentions a function
- ✅ **Version-specific lookups** - Need exact signature for your Appian version

**Workflow:**

**Step 1:** Read configured version from SKILL.md Configuration section.

**Step 2:** Check function existence (single command)

```bash
# Look up function (case-insensitive, follows redirects)
curl -sL "https://docs.appian.com/suite/help/26.6/functions.json" | jq -r '.["a!queryrecordtype"]'
# Returns: "/suite/help/26.6/fnc_system_queryrecordtype.html" if exists
# Returns: null if doesn't exist (DEFINITIVE)
```

**Step 3:** Fetch documentation page (if function exists and you need full details)

```bash
DOC_PATH=$(curl -sL "https://docs.appian.com/suite/help/26.6/functions.json" | jq -r '.["a!queryrecordtype"]')
if [ "$DOC_PATH" != "null" ]; then
  curl -sL "https://docs.appian.com$DOC_PATH"
fi
```

> **Performance tip:** For multiple lookups in one session, cache functions.json locally:
> ```bash
> curl -sL "https://docs.appian.com/suite/help/26.6/functions.json" > /tmp/appian-functions.json
> jq -r '.["functionname"]' /tmp/appian-functions.json
> ```

**Step 4:** Extract key information

- Function name and signature
- Parameters (name, type, required/optional, description)
- Return type
- Usage notes and examples
- Related functions

**Advantages over Tier 3:**
- ⚡ Faster (cached JSON lookup vs API call)
- 🎯 Exact match (no semantic ambiguity)
- ❌ Definitive negative results (function doesn't exist)
- 📌 Version-locked (not cross-version mixing)

**Fallback:** If configured version not found, default to 26.6 (latest).

**Failure handling:**
- If curl fails (network timeout, DNS failure, non-200 response): **skip Tier 3** (unreliable for function existence), go to best-effort
- If response is unexpected (HTML error page, empty body): **skip Tier 3**, go to best-effort
- No retries — one attempt, then proceed to best-effort
- Inform the user: "Could not verify function existence (network/documentation unavailable). Proceeding with skill references only."

**Why skip Tier 3 on Tier 2 failure:** Tier 3 has documented limitation—cannot definitively confirm function non-existence. Using Tier 3 after Tier 2 fails risks false positives (e.g., searching "regexmatch" returns "like()" function, incorrectly suggesting regexmatch exists).

**Success but incomplete documentation:**
- If Tier 2 confirms function exists but documentation page lacks practical examples:
  - Provide the signature/parameters found
  - Inform user: "Function exists (signature: X), but official docs lack detailed examples."
  - **Do NOT search Tier 3** for function usage examples (unreliable, may return unrelated content)
  - Provide best-effort example based on signature + Tier 1 patterns

**Version redirect handling:**
- `curl -sL` automatically follows redirects (e.g., 26.6 → 26.7 if 26.6 unavailable)
- If successful: use the redirected version (Appian maintains backward compatibility)
- Optionally inform user: "Note: Using functions.json from version 26.7 (26.6 redirected)"
- If redirect fails or loops: treat as network failure, skip Tier 3, go to best-effort

---

## Tier 3: Documentation Search Tool (Patterns & Recipes)

**What it provides:**
- Semantic search (understands natural language intent)
- UI/UX patterns and recipes
- Best practices and design guidelines
- Cross-version results (25.4, 26.3, etc.)

**Use for:**
- ✅ **UI/UX patterns** - "What is the drilldown pattern for grids?"
- ✅ **Recipes** - Query recipes, function recipes, interface recipes
- ✅ **"How to" questions** - "How do I filter a grid using a dropdown?"
- ✅ **Best practices** - "What are interface design best practices?"
- ✅ **Complex function examples** - When Tier 1/2 lack practical examples

**When to search:**
- User asks "how do I..." or "show me an example of..."
- Need patterns/recipes beyond skill references
- Skill references mention a topic but lack complete examples
- You would otherwise need to guess or invent the implementation

**Example natural language queries:**
- "What are the query recipes for filtering and aggregating records in Appian?"
- "Show me function recipes for working with arrays in Appian"
- "What are the interface recipes for building grids in Appian?"
- "What is the drilldown pattern for grids in Appian?"
- "Show me complete examples of a!queryRecordType with aggregations"

**Response format:**
- `source_url`: Direct URL to documentation section
- `content`: Focused snippet from specific documentation section (markdown)

**Limitations:**
- ❌ **Cannot definitively confirm non-existence** - May return related results even if exact function doesn't exist (e.g., searching "regexmatch" returns "like()" and pattern matching alternatives)
- ⚠️ **Quality depends on official docs** - If Appian docs lack practical examples, search won't find them
- 🌫️ **Semantic ambiguity** - May return broader results than exact match needed

**Failure modes:**

| Failure | Action |
|---------|--------|
| Authentication error | Note auth failure, skip Tier 3 for the rest of the session, proceed with Tier 1 + Tier 2 only |
| Tool not in tool list | Proceed without Tier 3, note documentation search is unavailable |
| Empty/irrelevant results | Inform user official docs lacked coverage for the query, proceed with skill references |

**When Tier 3 results mention functions:**
- **If function is critical to pattern implementation:** Verify existence via Tier 2 before using
  - Example: Tier 3 says "Use a!filterData() to filter grid results" → verify a!filterData() exists
  - Prevents implementing patterns with non-existent functions
- **If function is incidental/example only:** Trust Tier 3 content (don't over-verify)
  - Example: Tier 3 mentions "similar to a!forEach()" → no verification needed (just context)
- **Use judgment:** If uncertain whether function is critical, verify via Tier 2 (safe default)

**General rule:** When proceeding without Tier 3, inform the user what could not be verified and what assumptions were made.

**Rate limits:** 300 requests/day, 60 requests/minute (very generous).

---

## Which Tier to Use? Quick Reference

| Question Type | Use Tier | Example |
|---------------|----------|---------|
| Function exists? | **Tier 2** (functions.json) | "Does regexmatch() exist?" → definitive yes/no |
| Function signature? | **Tier 2** (functions.json) | "Parameters for a!queryRecordType()?" |
| Recipe? | **Tier 3** (search tool) | "Show me query recipes for filtering" |
| UI/UX pattern? | **Tier 3** (search tool) | "What is the drilldown pattern?" |
| "How to" question? | **Tier 3** (search tool) | "How to filter a grid by dropdown?" |

---

## Summary: Complementary Strengths

| Tier | Strength | Speed | Precision | Use Case |
|------|----------|-------|-----------|----------|
| **1: Skill Refs** | Curated examples, anti-patterns | ⚡⚡⚡ Instant | 🎯🎯🎯 Exact | First stop for all queries |
| **2: functions.json** | Existence checks, signatures | ⚡⚡ Fast | 🎯🎯🎯 Exact | Function verification |
| **3: Search Tool** | Recipes, patterns, best practices | ⚡ Slower | 🌫️ Semantic | Recipes, UI/UX patterns, "how to" |

**Always start with Tier 1.** Only proceed to Tier 2/3 when skill references lack sufficient detail.

---

## Examples (Extended)

### **Implementation Example 1: Expression rule with unknown function**

**Request:** "Create an expression rule that calculates distance in miles between two GPS coordinates"

**Step 1-2:** Load universal patterns + expression-rules.md

**Step 3 [CHECKPOINT]:**
- Check function-reference.md for distance functions → Not found
- Custom logic detected (distance calculation)
- **Automatically proceed to Tier 2** (special case: custom logic requires built-in check)
- Search functions.json for "distance" → Found `a!distanceBetween()`
- Fetch signature: `a!distanceBetween(startLatitude, startLongitude, endLatitude, endLongitude, unit)`

**Step 4:** No supplementary references needed

**Step 5:** Verify: function confirmed, null safety patterns loaded, types verified

**Step 6:** Implement using a!distanceBetween() with unit: "MILES"

**Result:** 5-line expression vs 40-line custom Haversine formula

---

### **Implementation Example 2: Interface with unknown pattern**

**Request:** "Create interface with grid that drills down to order details on row click"

**Step 1-2:** Load universal patterns + interfaces.md, sail.md

**Step 3 [CHECKPOINT]:**
- Check skill references for drilldown pattern → Not fully documented
- **Automatically proceed to Tier 3** (pattern not in Tier 1)
- Search "drilldown pattern grids Appian" → Found official recipe
- Pattern: Use local variable + conditional section (local!selectedRow + showWhen)

**Step 4:** No supplementary references needed

**Step 5:** Verify: pattern confirmed, components verified (a!gridField, a!sectionLayout)

**Step 6:** Implement using official drilldown pattern

**Result:** 1 validation error vs 4 errors when inventing (75% reduction)

---

### **Q&A Example 1: Function existence (anti-hallucination list)**

**Question:** "Does the function regexmatch() exist in Appian?"

**Tier 1 check:** function-reference.md anti-hallucination list → regexmatch() listed as NON-EXISTENT

**Answer:** "No, regexmatch() does not exist. Use like() or find() instead."

**Stop after answer** (no implementation requested)

---

### **Q&A Example 2: Function existence (not in Tier 1)**

**Question:** "Does a!flatten() exist in Appian?"

**Tier 1 check:** function-reference.md → Not listed  
**Tier 2 check:** Search functions.json → Found `a!flatten()`

**Answer:** "Yes, a!flatten() exists. It flattens nested arrays into a single-level array."

**Stop after answer** (no implementation requested)

---

### **Q&A Example 3: Function signature**

**Question:** "What are the parameters for a!startProcess()?"

**Tier 1 check:** function-reference.md → Not detailed  
**Tier 2 check:** Search functions.json → Found signature

**Answer:** "a!startProcess(processModel, processParameters, onSuccess, onError)"

**Stop after answer**

---

### **Q&A Example 4: Pattern lookup**

**Question:** "What is the drilldown pattern for grids in Appian?"

**Tier 1 check:** interfaces.md, sail.md → Basic grid info only  
**Tier 3 check:** Search "drilldown pattern grids" → Found official recipe

**Answer:** "Use local!selectedRow with a!dynamicLink() in grid column, then conditional a!sectionLayout() below grid that shows when local!selectedRow is not null."

**Stop after answer**

---

### **Q&A Example 5: Mixed query**

**Question:** "What are the parameters for a!gridField() and show me recipes for editable grids?"

**Tier 1 check:** function-reference.md → Not detailed  
**Tier 2 check:** Search functions.json for signature  
**Tier 3 check:** Search "editable grid recipes"

**Answer:** [Combine signature from Tier 2 + recipes from Tier 3]

**Stop after answer**

---

## Integration with SKILL.md Workflow

**This file supports SKILL.md Step 3 [MANDATORY CHECKPOINT]:**

1. **Step 3A (expression rules)** → Use Tier 2 workflow from this file
2. **Step 3B (interfaces)** → Use Tier 3 workflow from this file
3. **Error handling** → Apply failure mode guidance from Tier 2/3 sections

**Return to SKILL.md Step 4** after completing tier lookups.
