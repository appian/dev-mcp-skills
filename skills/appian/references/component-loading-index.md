# Component Loading Index

**Purpose:** Quick reference for loading SAIL component instruction files during interface development.

**When to use this file:**
- You're building an interface and need guidance for specific components
- Step 3B checkpoint redirects you here for instruction file mapping
- You need to know which components have detailed instruction files vs. registry-only coverage

---

## How to Use This Index

### During Interface Development (Step 3B Checkpoint)

1. **Check component existence** in `registry/components-registry.json`
   - If `exists: false` → STOP, use alternatives
   - If `exists: true` AND `hasInstructionFile: true` → Load the file below
   - If `exists: true` AND `hasInstructionFile: false` → Use Tier 2B (functions.json)

2. **Find your component** in the tables below

3. **Load the instruction file** for detailed guidance

---

## Components with Instruction Files (21 total)

### Layouts (9 components)

| Component | Instruction File | Key Guidance |
|---|---|---|
| **a!formLayout()** | `layouts/form-layout-instructions.md` | ContentsWidth defaults, button positioning, validation messages |
| **a!headerContentLayout()** | `layouts/header-content-layout-instructions.md` | Header/body structure, sticky headers, responsive behavior |
| **a!cardLayout()** | `layouts/card-layout-instructions.md` | Card structure, header/footer, hover states |
| **a!columnsLayout()** | `layouts/columns-layout-instructions.md` | Column distribution, responsive stacking, alignment |
| **a!sideBySideLayout()** | `layouts/sidebyside-layout-instructions.md` | Item alignment, spacing, vertical alignment options |
| **a!paneLayout()** | `layouts/pane-layout-instructions.md` | Panes structure, paneSplit ratios, responsive behavior |
| **a!tabLayout()** | `layouts/tab-layout-instructions.md` | Tab navigation, value/saveInto pattern, dynamic tabs |
| **a!wizardLayout()** | `layouts/wizard-layout-instructions.md` | Step sequence, validation, navigation between steps |
| **a!gridLayout()** | `components/grid-layout-instructions.md` | Row/column structure, responsive sizing, alignment |

### Selection & Input Components (2 components)

| Component | Instruction File | Key Guidance |
|---|---|---|
| **a!cardChoiceField()** | `components/card-choice-field-instructions.md` | Card-based selection, data structure, maxSelections |
| **a!buttonWidget()** | `components/button-instructions.md` | Button styles, loadingIndicator, confirmation patterns |

### Display Components (3 components)

| Component | Instruction File | Key Guidance |
|---|---|---|
| **a!imageField()** | `components/image-field-instructions.md` | Image sizing, altText for accessibility, document references |
| **a!richTextDisplayField()** | `components/rich-text-instructions.md` | Formatted text display, styling options, read-only patterns |
| **a!stampField()** | `components/stamp-field-instructions.md` | Status indicators, colors, icons, contentAlign |

### Grids (1 component)

| Component | Instruction File | Key Guidance |
|---|---|---|
| **a!gridField()** | `components/grid-field-instructions.md` | ⚠️ **CRITICAL:** showSearchBox/showRefreshButton/recordActions only work with recordType! data |

### Charts (6 components)

All chart components share the same instruction file with component-specific guidance:

| Component | Instruction File | Key Guidance |
|---|---|---|
| **a!areaChartField()** | `components/chart-instructions.md` | Area charts with shaded regions, trends over time |
| **a!barChartField()** | `components/chart-instructions.md` | Horizontal bars, category comparisons, stacking |
| **a!columnChartField()** | `components/chart-instructions.md` | Vertical bars, category comparisons, stacking |
| **a!lineChartField()** | `components/chart-instructions.md` | Line trends, multiple series, axis configuration |
| **a!pieChartField()** | `components/chart-instructions.md` | Circular slices, percentages, data labels |
| **a!scatterChartField()** | `components/chart-instructions.md` | 2D relationships, correlation patterns, axis titles |

---

## Components WITHOUT Instruction Files (~125 components)

For components not listed above, use this workflow:

1. **Check `registry/components-registry.json`** for existence and critical warnings
2. **Check `component-reference.md`** for signatures and parameters (~53 components documented)
3. **Use Tier 2B** (functions.json) for official documentation if needed
   - See `documentation-lookup-strategy.md` for Tier 2B workflow
   - Components are in functions.json alongside functions
4. **Use Tier 3** (doc search) for UI/UX patterns if needed

### Common Components Without Instruction Files

These have signatures in `component-reference.md` but no dedicated instruction files:

- **Inputs:** a!textField, a!paragraphField, a!dateField, a!dropdownField, a!checkboxField, a!radioButtonField, a!fileUploadField, a!pickerField
- **Layouts:** a!sectionLayout, a!columnLayout, a!boxLayout
- **Display:** a!textField (read-only), a!linkField, a!imageField, a!documentDownloadLink
- **Rich Text:** a!richTextBulletedList, a!richTextItem, a!richTextIcon, a!richTextLink

For these, component-reference.md provides signatures and Tier 2B provides official docs.

---

## Loading Strategy (Step 3B)

**Before implementing any interface:**

```
1. Identify all components you plan to use
2. For EACH component:
   a. Check registry/components-registry.json (existence + hasInstructionFile)
   b. If hasInstructionFile=true → Load from table above
   c. If hasInstructionFile=false → Check component-reference.md for signature
   d. If not in component-reference.md → Use Tier 2B (functions.json)
3. Review critical warnings from instruction files
4. Proceed with implementation
```

**Key instruction files to always consider:**
- **Layouts:** Start with form-layout-instructions.md (most common container)
- **Grids:** Load grid-field-instructions.md (critical warnings about recordType! data)
- **Charts:** Load chart-instructions.md (covers all 6 chart types)

---

## Cross-References

- **Component existence:** `registry/components-registry.json` (146 components)
- **Component signatures:** `references/component-reference.md` (~53 components)
- **Lookup workflows:** `references/documentation-lookup-strategy.md` (Tier 2B/3)
- **Anti-hallucination:** `component-reference.md` (components that DON'T exist)

---

## Maintenance

**When adding new instruction files:**
1. Add component to this index (appropriate category table)
2. Update `registry/components-registry.json` (set `hasInstructionFile: true`, add `instructionFile` path)
3. Update `component-reference.md` if component isn't already documented

**Current coverage:**
- 21 components with instruction files (this index)
- ~53 components with signatures in component-reference.md
- 146 total components in registry
- ~14% have instruction files, ~36% have markdown signatures, 100% in registry
