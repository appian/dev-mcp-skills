# SAIL GridField Usage Instructions

## Overview
GridField displays tabular data with built-in sorting, paging, selection, and search capabilities. It's optimized for read-only data presentation with rich formatting options for each column, including text, links, tags, buttons, and progress bars.

## ⚠️ CRITICAL: Record-Only Parameters

**These parameters ONLY work with record data sources and will cause errors with local data:**

- `showSearchBox` - ❌ **NEVER use with local data (a!map arrays)**
- `showRefreshButton` - ❌ **NEVER use with local data (a!map arrays)**
- `recordActions` - ❌ **NEVER use with local data (a!map arrays)**

### ❌ WRONG - Using record-only parameters with local data:
```sail
a!gridField(
  data: local!employees,  /* Local a!map array */
  columns: {...},
  showSearchBox: true,      /* ❌ ERROR: Only works with record data! */
  showRefreshButton: true   /* ❌ ERROR: Only works with record data! */
)
```

**Error:** These parameters require a record type data source and will fail at runtime when used with local variables.

### ✅ CORRECT - Omit these parameters for local data:
```sail
a!gridField(
  data: local!employees,  /* Local a!map array */
  columns: {...}
  /* ✅ Do NOT include showSearchBox or showRefreshButton */
)
```

### ✅ CORRECT - Use these parameters ONLY with record data:
```sail
a!gridField(
  data: recordType!Employee,  /* Record type data source */
  columns: {...},
  showSearchBox: true,         /* ✅ OK with record data */
  showRefreshButton: true      /* ✅ OK with record data */
)
```

**Key Takeaway:** If your grid uses `data: local!variableName` with a!map arrays, **NEVER** include `showSearchBox`, `showRefreshButton`, or `recordActions` parameters.

## Record-Powered Grid Features (Search, Filters, Export, Actions)

When a grid is backed by record data (`recordType!` or `a!recordData()`), it gains powerful built-in features that replace the need for custom search/filter UX. Understanding when to use custom UX vs built-in features is critical to avoid duplicate or conflicting interfaces.

### The Two Grid Modes

| Mode | Data Source | Search/Filter Approach |
|------|-------------|------------------------|
| **Mockup** | `local!` variables with `a!map()` | Custom UX (textField, dropdowns) with TODO comments |
| **Functional** | `recordType!` or `a!recordData()` | Built-in `showSearchBox`, `userFilters` |

**⚠️ CRITICAL:** Never mix custom search/filter UX with built-in grid features on the same grid. This creates duplicate, confusing interfaces.

### Built-in Features (Record Data Only)

When grid uses record data, these features are available:

| Feature | Parameter | Default | Description |
|---------|-----------|---------|-------------|
| Search box | `showSearchBox` | `true` | Full-text search across searchable record fields |
| Refresh button | `showRefreshButton` | `true` | Manual data refresh |
| Export button | `showExportButton` | `false` | Download grid data to Excel |
| User filters | `userFilters` | none | Dropdown and date range filters defined on record type |
| Record actions | `recordActions` | none | Action buttons above grid |

### Mockup Pattern: Custom Search for Local Data

When creating mockups with local data, implement a custom search field with a TODO comment:

```sail
/* =========================================================================
 * SEARCH AND FILTERS SECTION
 * TODO: When converting to record data, REMOVE this section and use:
 *   - showSearchBox: true (built-in search)
 *   - userFilters: { recordType!Record.filters.filterName, ... }
 * ========================================================================= */
local!searchText,
local!filterStatus,

/* Custom search field for mockup */
a!textField(
  label: "Search",
  labelPosition: "ABOVE",
  placeholder: "Search by name...",
  value: local!searchText,
  saveInto: local!searchText
),

/* Custom filter dropdown for mockup */
a!dropdownField(
  label: "Status",
  placeholder: "All Statuses",
  choiceLabels: {"Active", "Inactive", "Pending"},
  choiceValues: {"ACTIVE", "INACTIVE", "PENDING"},
  value: local!filterStatus,
  saveInto: local!filterStatus
),

/* Grid with manual filtering */
a!gridField(
  data: index(
    local!employees,
    wherecontains(
      true,
      a!forEach(
        items: local!employees,
        expression: and(
          or(
            a!isNullOrEmpty(local!searchText),
            search(lower(local!searchText), lower(fv!item.name)) > 0
          ),
          or(
            a!isNullOrEmpty(local!filterStatus),
            fv!item.status = local!filterStatus
          )
        )
      )
    ),
    {}
  ),
  columns: {...}
  /* ❌ NO showSearchBox - causes error with local data */
)
```

**Conversion Details:** The converter will transform custom search/filter UX to built-in grid features. See `/conversion-guidelines/display-conversion-grids.md` `{#display-grids.search-filter}` for:
- 4-step conversion process (inventory, match, apply decision matrix)
- Decision tree: when to convert vs keep custom UX
- User filter types (list filters, date range filters)
- TODO comment format for missing user filters
- Complete conversion examples

### When Custom Search/Filter UX is Appropriate

Keep custom search/filter sections when they serve **multiple components**, not just one grid:

```sail
/* ✅ APPROPRIATE: Dashboard filters affecting multiple grids AND charts */
a!cardLayout(
  contents: {
    /* These filters apply to the entire dashboard */
    a!dropdownField(label: "Region", ...),
    a!dateField(label: "Date Range Start", ...),
    a!dateField(label: "Date Range End", ...)
  }
),

/* Grid 1 - filtered by dashboard filters */
a!gridField(
  data: a!recordData(
    recordType: 'recordType!Sales',
    filters: {
      a!queryFilter(field: ..., value: local!filterRegion, applyWhen: ...),
      a!queryFilter(field: ..., value: local!filterDateStart, applyWhen: ...)
    }
  ),
  showSearchBox: true,  /* Grid-specific search still works */
  ...
),

/* Chart - also filtered by dashboard filters */
a!columnChartField(
  data: a!recordData(
    recordType: 'recordType!Sales',
    filters: {...same filters...}
  ),
  ...
)
```

**Important:** Dashboard-level filters remain as custom UX since they affect multiple components. Only single-grid filters are converted to built-in features.

### Record Actions (Toolbar and Row-Level)

**⚠️ NEVER use `recordActions` parameter with local data - causes runtime errors.**

When creating mockups with action buttons, use placeholder buttons with TODO comments for the converter to transform.

#### Mockup Pattern: Header/Toolbar Action Buttons

For Create or New actions, place buttons in the header or near the grid:

```sail
/* =========================================================================
 * HEADER WITH ACTION BUTTON
 * TODO: When converting to record data:
 *   - If action should stay in header: Convert to a!recordActionField()
 *   - If action can move to grid: Use recordActions parameter on a!gridField
 * ========================================================================= */
a!headerContentLayout(
  header: {
    a!sideBySideLayout(
      items: {
        a!sideBySideItem(
          item: a!richTextDisplayField(
            labelPosition: "COLLAPSED",
            value: a!richTextItem(text: "Memberships", size: "LARGE", style: "STRONG")
          )
        ),
        a!sideBySideItem(
          /* TODO: Convert to recordActions or a!recordActionField() */
          item: a!buttonArrayLayout(
            buttons: a!buttonWidget(
              label: "New Membership",
              icon: "plus",
              style: "SOLID",
              color: "ACCENT",
              saveInto: {}
            )
          ),
          width: "MINIMIZE"
        )
      }
    )
  },
  contents: {
    a!gridField(
      data: local!memberships,
      columns: {...}
      /* ❌ NO recordActions - causes error with local data */
    )
  }
)
```

**Conversion Details:** The converter will transform mockup action buttons to functional record actions. See `/conversion-guidelines/display-conversion-actions.md` for:
- Action type rules (Record List vs Related)
- Primary key identification
- Placement rules and validation
- refreshAfter parameter usage
- Style mapping and configuration

## ⚠️ CRITICAL: Function Variables in Grid Columns

**ONLY `fv!row` is available in grid columns!**

- ✅ `fv!row` - Access current row data
- ❌ `fv!index` - NOT AVAILABLE (causes runtime error)
- ❌ `fv!item` - NOT AVAILABLE (causes runtime error)

### ❌ COMMON MISTAKE: Using fv!index for Selection

**WRONG - This will fail:**
```sail
a!gridColumn(
  value: a!richTextItem(
    text: fv!row.name,
    link: a!dynamicLink(
      value: fv!index,  /* ERROR: fv!index doesn't exist in grid columns! */
      saveInto: local!selectedIndex
    )
  )
)
```

**RIGHT - Use grid's built-in selection:**
```sail
a!gridField(
  data: local!items,
  columns: { /* ... */ },
  selectable: true,
  selectionValue: local!selectedRows,  /* List of selected row data */
  selectionSaveInto: local!selectedRows,
  maxSelections: 1
)

/* Access selected item (selectionValue is always a LIST): */
local!firstSelected: index(local!selectedRows, 1, null)
```

## ⚠️ CRITICAL: Grid Selection Behavior - selectionValue Contains Identifiers

### Understanding selectionValue Content

**MOST CRITICAL RULE**: `selectionValue` does NOT contain full row objects!

When `selectable: true` is set on a gridField:
- `selectionValue` contains **identifiers from the data array**, NOT full row objects
- For static data using a!map(), `selectionValue` contains **integers** (the array indices - 1, 2, 3, etc.)
- For record data, `selectionValue` contains **record identifiers** (not full record data)

### ❌ CRITICAL ERROR: Accessing Properties on Selection Items

**WRONG - This will cause runtime error:**
```sail
a!gridField(
  data: local!courses,  /* Array of a!map() with id, name, type, etc. */
  selectable: true,
  selectionValue: local!selectedCourses,
  selectionSaveInto: local!selectedCourses,
  columns: {...}
)

/* Later in the code... */
a!forEach(
  items: local!selectedCourses,
  expression: fv!item.type  /* ❌ ERROR: fv!item is an integer, not a map! */
)
```

**Error Message You'll See:**
```
Expression evaluation error: Invalid index: Cannot index property 'type' of type Text into type Number (Integer)
```

**RIGHT - Look up full objects using index():**
```sail
a!gridField(
  data: local!courses,
  selectable: true,
  selectionValue: local!selectedCourses,  /* Contains integers like {1, 3, 5} */
  selectionSaveInto: local!selectedCourses,
  columns: {...}
)

/* ✅ Correct approach - look up full object using the identifier */
a!forEach(
  items: local!selectedCourses,
  expression: a!localVariables(
    local!course: index(local!courses, fv!item, a!map()),  /* fv!item is the integer ID */
    local!course.type  /* ✅ Now we can access properties */
  )
)
```

### Common Patterns for Working with Selected Data

**Pattern 1: Display selected items**
```sail
a!forEach(
  items: local!selectedCourses,
  expression: a!localVariables(
    local!course: index(local!availableCourses, fv!item, a!map()),
    a!richTextDisplayField(
      value: {
        a!richTextIcon(icon: "check-circle"),
        " ",
        local!course.name
      },
      labelPosition: "COLLAPSED"
    )
  )
)
```

**Pattern 2: Conditional logic based on selection**
```sail
showWhen: and(
  length(local!selectedCourses) > 0,
  or(
    a!forEach(
      items: local!selectedCourses,
      expression: index(local!availableCourses, fv!item, a!map()).type = "Contract"
    )
  )
)
```

**Pattern 3: Validation using selected data**
```sail
disableNextButton: or(
  length(local!selectedCourses) = 0,
  a!forEach(
    items: local!selectedCourses,
    expression: a!isNullOrEmpty(index(local!availableCourses, fv!item, a!map()).startDate)
  )
)
```

### Key Takeaways
- ✅ `selectionValue` contains **identifiers** (integers for static data, record IDs for records)
- ❌ `selectionValue` does NOT contain full row objects with properties
- ✅ Always use `index(dataArray, fv!item, defaultValue)` to look up full objects
- ✅ Use `a!localVariables(local!item: index(...), ...)` pattern for cleaner code
- ✅ Before iterating over selectionValue, ask: "Do I need to look up the full object?"

## ⚠️ CRITICAL GRID COLUMN RESTRICTIONS

### ❌ NEVER Use These in GridColumns:
- **sideBySideLayout** - Use rich text with icons/text instead
- **columnsLayout** - Not supported in grid cells
- **cardLayout** - Too complex for grid cells
- **Arrays of components** - Only single components allowed
- **a!textField** - Use plain text or a!richTextDisplayField instead

### ✅ ONLY Use These in GridColumns:
- Plain text values
- `a!richTextDisplayField` (for formatting, icons, multiple lines)
- `a!tagField` (for status indicators)
- `a!buttonArrayLayout` (for actions)
- `a!progressBarField` (for progress indicators)
- `a!imageField` (for small images/avatars)

## When to Use GridField vs Alternatives

### ✅ Use GridField For:
- **Read-only data tables** - Displaying lists of records, transactions, or reports

### ❌ Use GridLayout Instead For:
- **Editable tables** - When users need to edit data inline

### ❌ Use Alternative Components For:
- **Key-value pairs** - Use `a!sideBySideLayout`
- **Cards/tiles** - Use `a!cardGroupLayout` for non-tabular presentation

## Data Sources for Static Mockups

### Using a!map for Static Data
For mockups and prototypes, use `a!map()` to create sample data:

```sail
a!localVariables(
  local!claimsData: {
    a!map(
      id: "CLM-2024-001",
      dateFiled: date(2024, 3, 15),
      policyHolder: "John Smith", 
      vehicleInfo: "2019 Honda Civic",
      incidentType: "Collision",
      claimAmount: 8500,
      status: "Under Review",
      adjuster: "Sarah Johnson"
    ),
    a!map(
      id: "CLM-2024-002", 
      dateFiled: date(2024, 3, 12),
      policyHolder: "Maria Garcia",
      vehicleInfo: "2021 Toyota Camry", 
      incidentType: "Theft",
      claimAmount: 15000,
      status: "Approved",
      adjuster: "Mike Chen"
    )
  },
  
  a!gridField(
    data: local!claimsData,
    /* columns configuration */
  )
)
```

### Data Structure Requirements
- Each data item must be an `a!map()` with consistent field names
- Field names should match the `sortField` values in columns
- Use appropriate data types (dates, numbers, text)

## Core GridField Parameters

### Essential Parameters
- **data**: List of a!map() items for static mockups
- **columns**: Array of `a!gridColumn()` definitions
- **pageSize**: Number of rows per page (default: 10)
- **emptyGridMessage**: Text shown when no data exists

### Selection Parameters
- **selectable**: Boolean to enable row selection
- **selectionValue**: Currently selected row data
- **selectionSaveInto**: Variables updated on selection change
- **maxSelections**: Maximum allowed selections

### Visual Styling Parameters
- **spacing**: `STANDARD` (default) or `DENSE`
- **borderStyle**: `STANDARD` (default) or `LIGHT`
- **shadeAlternateRows**: Boolean for row striping (default: true)
- **height**: ⚠️ **ONLY set if grid needs fixed height with scrolling rows** - otherwise exclude or use `AUTO`

### Height Options (Only When Fixed Height is Needed)
```sail
/* Only use these when you need fixed height with scrolling */
height: "SHORT",           /* ~120px - very compact */
height: "SHORT_PLUS",      /* ~180px - compact */  
height: "MEDIUM",          /* ~240px - standard fixed height */
height: "MEDIUM_PLUS",     /* ~300px - expanded */
height: "TALL",            /* ~360px - large */
height: "TALL_PLUS",       /* ~480px - very large */
height: "EXTRA_TALL",      /* ~600px - maximum */

/* Default behavior - grid grows with content */
height: "AUTO"             /* Or omit height parameter entirely */
```
## GridColumn Configuration

### Essential Column Parameters
```sail
a!gridColumn(
  label: "Column Header",           /* Required: Column title */
  value: fv!row.fieldName,          /* Required: Cell content */
  width: "NARROW",                  /* Column width */
  sortField: "fieldName",           /* Enables sorting */
  align: "START",                   /* Cell alignment */
  backgroundColor: "#FEF2F2".     /* Optional cell background color for highlighting */
)
```

### ⚠️ CRITICAL: sortField Rules

**Each sortField value must be UNIQUE across all columns in a grid.**

#### The Fundamental Rule:
**sortField should match the primary field displayed in the column's value parameter.**

```sail
/* ✅ CORRECT - sortField matches the field being displayed */
a!gridColumn(
  label: "Partner Name",
  value: fv!row['recordType!SUBMISSION.relationships.partner.fields.name'],
  sortField: 'recordType!SUBMISSION.relationships.partner.fields.name'
)

/* ❌ WRONG - sortField doesn't match displayed field */
a!gridColumn(
  label: "Partner Name",
  value: fv!row['recordType!SUBMISSION.relationships.partner.fields.name'],
  sortField: 'recordType!SUBMISSION.fields.partnerUsername'  /* Different field! */
)
```

#### Common Mistake: Duplicate sortField Values
**Runtime Error:** If two columns use the same sortField, the interface will fail with:
```
Expression evaluation error: Each column must have a unique value for "sortField"
```

**Example of the problem:**
```sail
/* ❌ WRONG - Both columns use the same sortField */
a!gridColumn(
  label: "Partner Name",
  value: fv!row['...partner.fields.name'],
  sortField: 'recordType!SUBMISSION.fields.partnerUsername'  /* Duplicate! */
),
a!gridColumn(
  label: "Submitted By",
  value: fv!row['...partnerUsername'],
  sortField: 'recordType!SUBMISSION.fields.partnerUsername'  /* Duplicate! */
)
```

**✅ CORRECT Solution:**
```sail
/* ✅ Each column sorts by the field it displays */
a!gridColumn(
  label: "Partner Name",
  value: fv!row['...partner.fields.name'],
  sortField: 'recordType!SUBMISSION.relationships.partner.fields.name'  /* Unique */
),
a!gridColumn(
  label: "Submitted By",
  value: fv!row['...partnerUsername'],
  sortField: 'recordType!SUBMISSION.fields.partnerUsername'  /* Unique */
)
```

#### When to Omit sortField
If a column truly cannot have a unique sortField (rare edge case), omit the parameter entirely:
```sail
a!gridColumn(
  label: "Column Without Sorting",
  value: fv!row.complexCalculation
  /* No sortField - sorting disabled for this column */
)
```

### Column Width Guidelines
#### Strategy #1: grids with only a few columns
- Set all column widths to `AUTO`

#### Strategy #2: more complex spreadsheet-style grids
- Set each column width to a fixed value corresponding to content size:
    - `ICON`: Narrowest (~48px) - for single icons or indicators
    - `ICON_PLUS`: (~60px) - for short abbreviations
    - `NARROW`: (~82px) - very short numbers/text
    - `NARROW_PLUS`: (~172px) - short numbers/text
    - `MEDIUM`: (~260px) - names, short titles
    - `MEDIUM_PLUS`: (~430px) - short descriptions
    - `WIDE`: Large content (~600px) - long descriptions/paragraphs

❌❌❌ `WIDE_PLUS` is NOT a valid column width!!!

### Column Alignment Options
- **START**: Left-aligned (default)
- **CENTER**: Center-aligned
- **END**: Right-aligned (ideal for numbers, amounts)

## Common Column Content Patterns

### 1. Basic Text with Styling
```sail
a!gridColumn(
  label: "Policy Holder",
  value: fv!row.policyHolder,
  width: "NARROW_PLUS",
  sortField: "policyHolder"
)
```

### 2. Linked Text (Clickable Records)

**Best Practice:** Make the **first text field** (name, title, number) clickable for record navigation, not numeric IDs. Keep the ID in the data and in the dynamicLink value parameter for converter reference.

```sail
a!gridColumn(
  label: "Claim Title",
  value: a!richTextDisplayField(
    value: a!richTextItem(
      text: fv!row.title,
      style: "STRONG",
      /* TODO-CONVERTER: Convert to a!recordLink() using record type primary key */
      link: a!dynamicLink(value: fv!row.id, saveInto: {}),
      linkStyle: "STANDALONE"
    ),
    labelPosition: "COLLAPSED"
  ),
  width: "MEDIUM",
  sortField: "title"
)
```

### 3. Formatted Currency/Numbers
```sail
a!gridColumn(
  label: "Claim Amount",
  value: a!richTextDisplayField(
    value: a!richTextItem(
      text: dollar(fv!row.claimAmount),
      style: "STRONG"
    ),
    labelPosition: "COLLAPSED"
  ),
  width: "NARROW",
  align: "END",
  sortField: "claimAmount"
)
```

### 4. Status Tags
```sail
a!gridColumn(
  label: "Status",
  value: a!tagField(
    tags: a!tagItem(
      text: fv!row.status,
      backgroundColor: if(
        fv!row.status = "Approved",
        "POSITIVE",
        if(
          fv!row.status = "Denied",
          "NEGATIVE", 
          if(
            fv!row.status = "Under Review",
            "ACCENT",
            "SECONDARY"
          )
        )
      )
    ),
    size: "SMALL",
    labelPosition: "COLLAPSED"
  ),
  width: "NARROW_PLUS",
  sortField: "status"
)
```

### 5. Stacked Content (Multiple Lines per Cell)

```sail
a!gridColumn(
  label: "Claim Details",
  value: a!richTextDisplayField(
    value: {
      a!richTextItem(
        text: fv!row.id,
        style: "STRONG"
      ),
      char(10),
      a!richTextItem(
        text: text(fv!row.dateFiled, "MMM d, yyyy"),
        color: "SECONDARY",
        size: "SMALL"
      )
    },
    labelPosition: "COLLAPSED"
  ),
  width: "NARROW_PLUS",
  sortField: "id"
)
```

### 6. Icons with Text
**❌ WRONG - Using sideBySideLayout:**
```sail
a!gridColumn(
  value: a!sideBySideLayout(    /* ❌ NOT ALLOWED IN GRID COLUMNS */
    items: {
      a!sideBySideItem(item: a!richTextIcon(icon: "user")),
      a!sideBySideItem(item: a!richTextDisplayField(...))
    }
  )
)
```

**✅ CORRECT - Using richTextDisplayField with icons:**
```sail
a!gridColumn(
  label: "Contact Info",
  value: a!richTextDisplayField(
    value: {
      a!richTextIcon(icon: "user", size: "SMALL", color: "#6B7280"),
      " ",
      fv!row.policyHolder,
      char(10),
      a!richTextIcon(icon: "phone", size: "SMALL", color: "#6B7280"),
      " ",
      a!richTextItem(
        text: fv!row.phoneNumber,
        color: "SECONDARY",
        size: "SMALL"
      )
    },
    labelPosition: "COLLAPSED"
  ),
  width: "MEDIUM",
  sortField: "policyHolder"
)
```

### 7. Action Buttons

**⚠️ Important:** Only use action buttons in grid columns for **secondary actions** (like Delete, Duplicate, Share). For the primary action of viewing a record, use a link on the row identifier/display name instead.

**Mockup Pattern:**
```sail
a!gridColumn(
  label: "Actions",
  value: a!buttonArrayLayout(
    buttons: {
      /* TODO-CONVERTER: Convert to a!recordActionField() with identifier */
      a!buttonWidget(
        label: "Delete",
        icon: "trash",
        style: "OUTLINE",
        size: "SMALL",
        color: "SECONDARY",
        saveInto: {}
      ),
      a!buttonWidget(
        label: "Share",
        icon: "share",
        style: "OUTLINE",
        size: "SMALL",
        color: "SECONDARY",
        saveInto: {}
      )
    },
    align: "START"
  ),
  width: "NARROW",
  align: "CENTER"
)
```

**Notes:**
- Use `a!buttonArrayLayout` to wrap multiple buttons in grid columns
- Empty `saveInto: {}` indicates this is a mockup placeholder
- Converter will transform to `a!recordActionField()` with proper action references
- See `/conversion-guidelines/display-conversion-actions.md` for conversion patterns

### 8. Conditional Background Color Highlighting

To conditionally set cell, row, or column background color, use `if` of `a!match` to set backgroundColor property on `a!gridColumn`

```sail
a!gridField(
  data: local!taskData,
  columns: {
    a!gridColumn(
      label: "Task Name",
      value: fv!row.taskName,
      width: "MEDIUM"
    ),
    a!gridColumn(
      label: "Due Date",
      value: text(fv!row.dueDate, "MMM d, yyyy"),
      width: "NARROW",
      backgroundColor: if( /* Set red background color if overdue */
        fv!row.dueDate < today(),
        "ERROR",
        "NONE"
      )
    ),
    a!gridColumn(
      label: "Assigned To",
      value: fv!row.assignedTo,
      width: "NARROW"
    )
  }
)
```

## Advanced Grid Patterns

### Standard Grid
```sail
a!gridField(
  data: local!claimsData,
  columns: {
    a!gridColumn(
      label: "Claim",
      value: a!richTextDisplayField(
        value: {
          a!richTextItem(
            text: fv!row.title,
            style: "STRONG",
            /* TODO-CONVERTER: Convert to a!recordLink() using record type primary key */
            link: a!dynamicLink(value: fv!row.id, saveInto: {}),
            linkStyle: "STANDALONE"
          ),
          char(10),
          a!richTextItem(
            text: text(fv!row.dateFiled, "MMM d, yyyy"),
            color: "SECONDARY",
            size: "SMALL"
          )
        },
        labelPosition: "COLLAPSED"
      ),
      width: "MEDIUM",
      sortField: "title"
    ),
    a!gridColumn(
      label: "Insured",
      value: a!richTextDisplayField(
        value: {
          a!richTextIcon(icon: "user", size: "SMALL", color: "#6B7280"),
          " ",
          fv!row.policyHolder,
          char(10),
          a!richTextIcon(icon: "car", size: "SMALL", color: "#6B7280"),
          " ",
          a!richTextItem(
            text: fv!row.vehicleInfo,
            color: "SECONDARY",
            size: "SMALL"
          )
        },
        labelPosition: "COLLAPSED"
      ),
      width: "MEDIUM",
      sortField: "policyHolder"
    ),
    a!gridColumn(
      label: "Claim Amount",
      value: a!richTextDisplayField(
        value: {
          a!richTextItem(
            text: dollar(fv!row.claimAmount),
            style: "STRONG",
            size: "MEDIUM"
          ),
          char(10),
          a!richTextItem(
            text: fv!row.incidentType,
            color: "SECONDARY",
            size: "SMALL"
          )
        },
        labelPosition: "COLLAPSED"
      ),
      width: "NARROW_PLUS",
      align: "END",
      sortField: "claimAmount"
    ),
    a!gridColumn(
      label: "Status",
      value: a!tagField(
        tags: a!tagItem(
          text: fv!row.status,
          backgroundColor: if(
            fv!row.status = "Approved",
            "POSITIVE",
            if(
              fv!row.status = "Denied",
              "NEGATIVE",
              if(
                fv!row.status = "Under Review",
                "ACCENT",
                "SECONDARY"
              )
            )
          )
        ),
        size: "SMALL",
        labelPosition: "COLLAPSED"
      ),
      width: "NARROW_PLUS",
      sortField: "status"
    )
  },
  pageSize: 15,
  initialSorts: {
    a!sortInfo(field: "dateFiled", ascending: false)
  },
  spacing: "STANDARD",
  borderStyle: "LIGHT",
  shadeAlternateRows: true,
  emptyGridMessage: "No claims found matching your criteria.",
  selectable: true,
  selectionValue: local!selectedClaims,
  selectionSaveInto: local!selectedClaims,
  maxSelections: 5
  /* No height parameter - grid grows with content */
)
```

## Sorting and Paging Configuration

### Initial Sorting
```sail
a!gridField(
  initialSorts: {
    a!sortInfo(field: "dateFiled", ascending: false),
    a!sortInfo(field: "claimAmount", ascending: false)
  },
  secondarySorts: {
    a!sortInfo(field: "policyHolder", ascending: true)
  }
)
```

### Paging Configuration
```sail
a!gridField(
  pageSize: 25,                    /* Rows per page */
  pagingSaveInto: local!gridPaging /* Track paging state */
)
```

## Visual Styling Options

### Grid Appearance (Auto Height)
```sail
a!gridField(
  spacing: "DENSE",              /* Compact row height */
  borderStyle: "LIGHT",          /* Subtle borders */
  shadeAlternateRows: false,     /* No row striping */
  /* No height parameter - grid grows with content */
)
```

### Grid Appearance (Fixed Height for Scrolling)
```sail
a!gridField(
  spacing: "DENSE",              /* Compact row height */
  borderStyle: "LIGHT",          /* Subtle borders */
  shadeAlternateRows: false,     /* No row striping */
  height: "MEDIUM"               /* Fixed height with scrollable rows */
)
```

### Selection Styling
```sail
a!gridField(
  selectable: true,
  selectionStyle: "CHECKBOX",     /* or "ROW_HIGHLIGHT" */
  showSelectionCount: true,       /* Display selection counter */
  maxSelections: 10
)
```

## Best Practices

### ✅ DO:
- Use appropriate column widths based on content type
- Provide meaningful `emptyGridMessage` text
- Use consistent text styling across similar columns
- **Set `sortField` to match the primary field displayed in the column's value parameter**
- Use `align: "END"` for numeric columns
- Choose appropriate `pageSize` for your use case (10-25 typical)
- Use `a!richTextDisplayField` for formatted content instead of sideBySideLayout
- **Exclude height parameter unless you need fixed height with scrolling**

### ❌ DON'T:
- **NEVER use `a!sideBySideLayout` inside grid columns** - Use multiple rich text items instead
- Put `a!columnsLayout` or `a!cardLayout` inside grid columns
- **Set height parameter unless you specifically need fixed height with scrolling**
- Use very wide grids with too many columns (consider stacking content)
- Forget to set column widths (can cause layout issues)
- **Use the same sortField value in multiple columns** - Each must be unique AND match the displayed field ‼️
- **Add sortField to computed columns (if/a!match/concat)** - Only raw field displays are sortable ‼️

### Column Content Guidelines:
- **IDs/References**: Use linked rich text with `linkStyle: "STANDALONE"`
- **Names/Text**: Plain text or rich text with emphasis
- **Numbers/Currency**: Right-aligned rich text with formatting
- **Dates**: Formatted text with consistent date format
- **Status**: Tag fields with semantic colors
- **Actions**: Button arrays with consistent styling
- **Icons + Text**: Use `a!richTextDisplayField` with `a!richTextIcon()` and `a!richTextItem()`

## Common Validation Issues

### ❌ WRONG - Using record-only parameters with local data:
```sail
a!gridField(
  data: local!topOrgs,  /* ❌ Local data, not record data */
  columns: {...},
  showSearchBox: false,       /* ❌ ERROR: Don't use with local data */
  showRefreshButton: false    /* ❌ ERROR: Don't use with local data */
)
```

### ✅ CORRECT - Omit record-only parameters:
```sail
a!gridField(
  data: local!topOrgs,
  columns: {...}
  /* ✅ No showSearchBox or showRefreshButton */
)
```

### ❌ WRONG - Using sideBySideLayout in grid column:
```sail
a!gridColumn(
  value: a!sideBySideLayout(     /* ❌ NOT ALLOWED IN GRID COLUMNS */
    items: {
      a!sideBySideItem(item: a!richTextIcon(...)),
      a!sideBySideItem(item: a!richTextDisplayField(...))
    }
  )
)
```

### ✅ CORRECT - Using richTextDisplayField:
```sail
a!gridColumn(
  value: a!richTextDisplayField(
    value: {
      a!richTextIcon(icon: "user", size: "SMALL"),
      " ",
      a!richTextItem(text: fv!row.name, style: "STRONG")
    },
    labelPosition: "COLLAPSED"
  )
)
```

## ⚠️ CRITICAL VALIDATION CHECKLIST

### Grid Parameters Validation:
- [ ] ❌ If using local data (`data: local!variable`), **NO** `showSearchBox` parameter ‼️
- [ ] ❌ If using local data (`data: local!variable`), **NO** `showRefreshButton` parameter ‼️
- [ ] ❌ If using local data (`data: local!variable`), **NO** `recordActions` parameter ‼️
- [ ] ✅ Record-only parameters used ONLY with `recordType!` or `a!recordData()` ‼️

### Grid Column Content Validation:
- [ ] ❌ **NO sideBySideLayouts** in any grid column
- [ ] ❌ **NO columnsLayouts** in any grid column
- [ ] ❌ **NO cardLayouts** in any grid column
- [ ] ❌ **NO arrays of components** in any grid column
- [ ] ✅ **ONLY** single components: richTextDisplayField, tagField, buttonArrayLayout, etc.
- [ ] ✅ richTextItem must be inside richTextDisplayField

### Alternative Solutions:
- **Instead of sideBySideLayout**: Use `a!richTextDisplayField` with `a!richTextIcon()` and `a!richTextItem()`
- **Instead of complex layouts**: Break content into multiple columns or use stacked rich text
- **For multiple actions**: Use `a!buttonArrayLayout` with multiple buttons
- **Instead of fixed height**: Let grid grow naturally unless specifically constrained

### Grid Column Widths Validation:
- [ ] ✅ All columns have `AUTO` width - OR - all columns have fixed widths (e.g., `MEDIUM`)
- [ ] ❌ NO invalid width values like `WIDE_PLUS`
