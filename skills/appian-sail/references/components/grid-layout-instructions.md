# SAIL GridLayout (Editable Grid) Usage Instructions

## Overview
GridLayout (`a!gridLayout`) displays tabular layouts of SAIL components for **inline editing** of fields. Unlike `a!gridField` which is read-only, `a!gridLayout` allows users to directly edit data within the grid rows. It's designed for structured data entry and editing scenarios.

## ⚠️ CRITICAL: GridLayout vs GridField

### ✅ Use a!gridLayout (Editable Grid) For:
- **Inline editing** - Users need to modify data directly in the table
- **Bulk data entry** - Multiple records created/updated at once
- **Form inputs in rows** - Each row contains editable fields
- **Add/remove rows functionality** - Dynamic row management

### ❌ Use a!gridField (Read-only Grid) Instead For:
- **Read-only data display** - Simple data presentation
- **Record lists** - Displaying existing records without editing
- **Reports and dashboards** - Static data views

## Core GridLayout Parameters

### Essential Configuration
```sail
a!gridLayout(
  label: "Grid Label",
  headerCells: {
    a!gridLayoutHeaderCell(label: "Column 1"),
    a!gridLayoutHeaderCell(label: "Column 2")
  },
  columnConfigs: {
    a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 1),
    a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 2)
  },
  rows: {
    a!gridRowLayout(
      contents: {
        a!textField(value: "Cell 1", saveInto: local!data[1].field1),
        a!textField(value: "Cell 2", saveInto: local!data[1].field2)
      }
    )
  }
)
```

### Key Parameters

**Essential:**
- **label** (Text): Grid label display
- **headerCells** (List of GridLayoutHeaderCell): Column headers using `a!gridLayoutHeaderCell()`
- **columnConfigs** (List of GridLayoutColumnConfig): Column width configuration using `a!gridLayoutColumnConfig()`
- **rows** (List of GridRowLayout): Grid rows created with `a!gridRowLayout()`

**Display & Styling:**
- **instructions** (Text): Supplemental guidance below label
- **emptyGridMessage** (Text): Message when no rows exist (default: "No items available")
- **height** (Text): `"SHORT"`, `"MEDIUM"`, `"TALL"`, `"AUTO"` (default)
- **spacing** (Text): `"STANDARD"` (default) or `"DENSE"`
- **borderStyle** (Text): `"STANDARD"` (default) or `"LIGHT"`
- **shadeAlternateRows** (Boolean): Alternate row shading (default: true)

**Selection:**
- **selectable** (Boolean): Enable row selection column (default: false)
- **selectionValue** (List of Variant): Selected row identifiers
- **selectionSaveInto** (List of Save): Variables updated on selection
- **selectionStyle** (Text): `"CHECKBOX"` (default) or `"ROW_HIGHLIGHT"`
- **selectionDisabled** (Boolean): Disable all selection
- **selectionRequired** (Boolean): Require selection for submission

**Validation:**
- **validations** (List of Variant): Error messages displayed below grid
- **validationGroup** (Text): Group name for conditional validation (no spaces)

**Accessibility:**
- **labelPosition** (Text): `"ABOVE"` (default), `"ADJACENT"`, `"COLLAPSED"`, `"JUSTIFIED"`
- **rowHeader** (Number Integer): Index of column to use as row header for screen readers (1-indexed)

**Advanced:**
- **addRowLink** (Link): Link to add new rows (e.g., `a!dynamicLink()`)
- **rowDeletions** (List of GridRowDeletion): Row deletion configurations using `a!gridRowDeletion()`
- **totalCount** (Number Integer): Total row count for pagination display

## GridLayout Header Cells

Define column headers with `a!gridLayoutHeaderCell()`:

```sail
a!gridLayoutHeaderCell(
  label: "Employee Name",
  helpTooltip: "Enter first and last name",
  align: "LEFT"  /* LEFT (default), CENTER, RIGHT */
)
```

**Parameters:**
- **label** (Text): Column header text
- **helpTooltip** (Text): Tooltip on hover
- **align** (Text): `"LEFT"` (default), `"CENTER"`, `"RIGHT"`
- **showWhen** (Boolean): Whether to display this header cell (default: true)

## GridLayout Column Configuration

### Column Width Strategies

**Strategy 1: Distribute/Proportional Widths (Few Columns)**
Use weighted proportions to distribute available space if the grid only has a few columns and horizontal scrolling is unlikely:

```sail
columnConfigs: {
  a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 2),  /* 40% of space */
  a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 2),  /* 40% of space */
  a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 1)   /* 20% of space */
}
```

**Strategy 2: Fixed Width (Spreadsheet-Style)**
Set specific pixel widths for columns to match the size of column contents (like a spreadsheet):

```sail
columnConfigs: {
  a!gridLayoutColumnConfig(width: "NARROW"),      /* ~100px */
  a!gridLayoutColumnConfig(width: "NARROW_PLUS"), /* ~150px */
  a!gridLayoutColumnConfig(width: "MEDIUM"),      /* ~200px */
  a!gridLayoutColumnConfig(width: "WIDE")         /* ~400px */
}
```

**Valid Width Values:**
- `"DISTRIBUTE"`: Proportional width based on weight (default)
- `"ICON"`: ~48px - for icon-only columns
- `"ICON_PLUS"`: ~60px - for small indicators
- `"NARROW"`: ~100px - short text/numbers
- `"NARROW_PLUS"`: ~150px - IDs, short codes
- `"MEDIUM"`: ~200px - names, standard fields
- `"MEDIUM_PLUS"`: ~300px - longer text fields
- `"WIDE"`: ~400px - descriptions, notes

**Strategy 3: Mixed Width**
Combine fixed and distributed widths:

```sail
columnConfigs: {
  a!gridLayoutColumnConfig(width: "ICON"),                   /* Fixed icon column */
  a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 3),  /* 60% of remaining */
  a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 2),  /* 40% of remaining */
  a!gridLayoutColumnConfig(width: "NARROW")                  /* Fixed action column */
}
```

**a!gridLayoutColumnConfig Parameters:**
- **width** (Text): Column width - `"DISTRIBUTE"` (default), `"ICON"`, `"ICON_PLUS"`, `"NARROW"`, `"NARROW_PLUS"`, `"MEDIUM"`, `"MEDIUM_PLUS"`, `"WIDE"`
- **weight** (Number Integer): For DISTRIBUTE width, relative proportion from 1-10 (default: 1)
- **showWhen** (Boolean): Whether to display this column (default: true)

### Column Content Alignment

**Note:** Column alignment is controlled by the **header cell**, not the column config. Set alignment in `a!gridLayoutHeaderCell()`:

```sail
headerCells: {
  a!gridLayoutHeaderCell(label: "Amount", align: "RIGHT"),
  a!gridLayoutHeaderCell(label: "Name", align: "LEFT"),
  a!gridLayoutHeaderCell(label: "Status", align: "CENTER")
}
```

**Alignment Best Practices:**
- **Text/names**: LEFT alignment (default)
- **Numbers/currency**: RIGHT alignment
- **Status indicators**: CENTER alignment
- **Action buttons**: CENTER or RIGHT alignment

## GridLayout Rows

Create rows with `a!gridRowLayout()`:

```sail
a!gridRowLayout(
  contents: {
    a!textField(
      value: fv!item.name,
      saveInto: fv!item.name,
      required: true,
      labelPosition: "COLLAPSED"
    ),
    a!dateField(
      value: fv!item.startDate,
      saveInto: fv!item.startDate,
      labelPosition: "COLLAPSED"
    ),
    a!dropdownField(
      choiceLabels: {"Active", "Inactive"},
      choiceValues: {"active", "inactive"},
      value: fv!item.status,
      saveInto: fv!item.status,
      labelPosition: "COLLAPSED"
    )
  },
  id: fv!item.id  /* Unique identifier for row selection */
)
```

**a!gridRowLayout Parameters:**
- **contents** (List of Variant): Array of components to display in grid cells. Supported: Barcode, Checkboxes, Date, Date & Time, Dropdown, Floating Point, File Upload, Image, Integer, Link, Multiple Dropdown, Paragraph, Pickers, Progress Bar, Radio Buttons, Rich Text Display, and Text.
- **id** (Any Type): Value that uniquely identifies this row for selection purposes
- **selectionDisabled** (Boolean): When selection is enabled, disables selection for this specific row (default: false)
- **showWhen** (Boolean): Whether to display this row (default: true)

### Row Contents Requirements

**✅ Allowed Components in GridLayout Rows:**
- `a!textField()` - Single line text input
- `a!paragraphField()` - Multi-line text input
- `a!integerField()` - Number input
- `a!floatingPointField()` - Floating point number input
- `a!dateField()` - Date picker
- `a!dateTimeField()` - Date and time picker
- `a!dropdownField()` - Dropdown selection
- `a!checkboxField()` - Checkbox selection
- `a!radioButtonField()` - Radio buttons (use sparingly)
- `a!richTextDisplayField()` - Read-only formatted text
- `a!buttonArrayLayout()` - Action buttons
- `a!imageField()` - Small images/avatars

**❌ NOT ALLOWED in GridLayout Rows:**
- `a!sideBySideLayout()`
- `a!columnsLayout()`
- `a!cardLayout()`
- Nested layouts

### Common Row Patterns

#### Pattern 1: Basic Text Input Row
```sail
a!forEach(
  items: local!employees,
  expression: a!gridRowLayout(
    contents: {
      a!textField(
        value: fv!item.firstName,
        saveInto: fv!item.firstName,
        labelPosition: "COLLAPSED",
        required: true
      ),
      a!textField(
        value: fv!item.lastName,
        saveInto: fv!item.lastName,
        labelPosition: "COLLAPSED",
        required: true
      ),
      a!textField(
        value: fv!item.email,
        saveInto: fv!item.email,
        labelPosition: "COLLAPSED",
        validations: if(
          a!isNullOrEmpty(fv!item.email),
          {},
          if(
            and(
              not(contains(fv!item.email, "@")),
              not(contains(fv!item.email, "."))
            ),
            "Please enter a valid email address",
            {}
          )
        )
      )
    },
    id: fv!index
  )
)
```

#### Pattern 2: Mixed Input Types with Dropdowns
```sail
a!forEach(
  items: local!tasks,
  expression: a!gridRowLayout(
    contents: {
      a!textField(
        value: fv!item.taskName,
        saveInto: fv!item.taskName,
        labelPosition: "COLLAPSED"
      ),
      a!dropdownField(
        choiceLabels: {"High", "Medium", "Low"},
        choiceValues: {"high", "medium", "low"},
        value: fv!item.priority,
        saveInto: fv!item.priority,
        labelPosition: "COLLAPSED",
        placeholder: "Select priority"
      ),
      a!dateField(
        value: fv!item.dueDate,
        saveInto: fv!item.dueDate,
        labelPosition: "COLLAPSED"
      ),
      a!buttonArrayLayout(
        buttons: {
          a!buttonWidget(
            label: "Delete",
            icon: "trash",
            style: "OUTLINE",
            size: "SMALL",
            color: "SECONDARY",
            value: fv!index,
            saveInto: a!save(
              local!tasks,
              remove(local!tasks, save!value)
            )
          )
        },
        align: "CENTER"
      )
    },
    id: fv!index
  )
)
```

#### Pattern 3: Row with Validation and Status Display
```sail
a!forEach(
  items: local!orders,
  expression: a!gridRowLayout(
    contents: {
      a!richTextDisplayField(
        value: a!richTextItem(
          text: fv!item.orderId,
          style: "STRONG"
        ),
        labelPosition: "COLLAPSED"
      ),
      a!integerField(
        value: fv!item.quantity,
        saveInto: fv!item.quantity,
        labelPosition: "COLLAPSED",
        validations: if(
          or(
            a!isNullOrEmpty(fv!item.quantity),
            fv!item.quantity < 1
          ),
          "Quantity must be at least 1",
          {}
        )
      ),
      a!floatingPointField(
        value: fv!item.unitPrice,
        saveInto: fv!item.unitPrice,
        labelPosition: "COLLAPSED"
      ),
      a!richTextDisplayField(
        value: a!richTextItem(
          text: dollar(fv!item.quantity * fv!item.unitPrice),
          style: "STRONG"
        ),
        labelPosition: "COLLAPSED"
      )
    },
    id: fv!item.orderId
  )
)
```

## Row Selection Configuration

Enable users to select rows for batch operations:

```sail
a!gridLayout(
  label: "Employee List",
  selectable: true,
  selectionValue: local!selectedEmployees,
  selectionSaveInto: local!selectedEmployees,
  selectionStyle: "CHECKBOX",  /* or "ROW_HIGHLIGHT" */
  selectionRequired: false,
  rows: /* row configuration */
)
```

**Selection Parameters:**
- **selectable**: Must be `true` to enable selection
- **selectionValue**: Variable storing selected row IDs
- **selectionSaveInto**: Save target for selection changes
- **selectionStyle**: `"CHECKBOX"` (shows checkboxes) or `"ROW_HIGHLIGHT"` (click entire row)
- **selectionRequired**: If `true`, user must select at least one row before submitting

## Adding and Deleting Rows

### Adding Rows with a!dynamicLink

```sail
a!localVariables(
  local!employees: {
    a!map(firstName: "", lastName: "", email: "", id: 1)
  },
  local!idCounter: 2,

  a!gridLayout(
    label: "Employees",
    headerCells: {
      a!gridLayoutHeaderCell(label: "First Name"),
      a!gridLayoutHeaderCell(label: "Last Name"),
      a!gridLayoutHeaderCell(label: "Email"),
      a!gridLayoutHeaderCell(label: "Actions")
    },
    columnConfigs: {
      a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 1),
      a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 1),
      a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 2),
      a!gridLayoutColumnConfig(width: "NARROW")
    },
    rows: a!forEach(
      items: local!employees,
      expression: a!gridRowLayout(
        contents: {
          a!textField(
            value: fv!item.firstName,
            saveInto: fv!item.firstName,
            labelPosition: "COLLAPSED"
          ),
          a!textField(
            value: fv!item.lastName,
            saveInto: fv!item.lastName,
            labelPosition: "COLLAPSED"
          ),
          a!textField(
            value: fv!item.email,
            saveInto: fv!item.email,
            labelPosition: "COLLAPSED"
          ),
          a!buttonArrayLayout(
            buttons: {
              a!buttonWidget(
                label: "Delete",
                icon: "trash",
                style: "OUTLINE",
                size: "SMALL",
                color: "SECONDARY",
                value: fv!index,
                saveInto: a!save(
                  local!employees,
                  remove(local!employees, save!value)
                )
              )
            }
          )
        },
        id: fv!item.id
      )
    ),
    addRowLink: a!dynamicLink(
      label: "Add Employee",
      value: a!map(firstName: "", lastName: "", email: "", id: local!idCounter),
      saveInto: {
        a!save(local!employees, append(local!employees, save!value)),
        a!save(local!idCounter, local!idCounter + 1)
      }
    ),
    emptyGridMessage: "No employees added. Click 'Add Employee' to begin."
  )
)
```

### Row Deletions with a!gridRowDeletion

Alternative approach using built-in row deletion:

```sail
a!gridLayout(
  rows: a!forEach(
    items: local!items,
    expression: a!gridRowLayout(
      contents: { /* row contents */ },
      id: fv!item.id
    )
  ),
  rowDeletions: {
    a!gridRowDeletion(
      icon: "trash",
      link: a!dynamicLink(
        saveInto: a!save(
          local!items,
          remove(local!items, wherecontains(save!value, local!items.id))
        )
      ),
      isVisible: true
    )
  }
)
```

## Data Structure for Static Mockups

Use `a!map()` arrays for prototype data:

```sail
a!localVariables(
  local!projectTasks: {
    a!map(
      id: 1,
      taskName: "Design wireframes",
      assignedTo: "Sarah Johnson",
      priority: "high",
      dueDate: today() + 7,
      status: "in_progress",
      hoursEstimated: 16
    ),
    a!map(
      id: 2,
      taskName: "API development",
      assignedTo: "Mike Chen",
      priority: "high",
      dueDate: today() + 14,
      status: "not_started",
      hoursEstimated: 40
    ),
    a!map(
      id: 3,
      taskName: "User testing",
      assignedTo: "Lisa Park",
      priority: "medium",
      dueDate: today() + 21,
      status: "not_started",
      hoursEstimated: 8
    )
  },

  a!gridLayout(
    /* grid configuration */
  )
)
```

### Data Structure Requirements
- Each item must be `a!map()` with consistent fields
- Include an `id` field for row identification
- Use appropriate data types (dates, numbers, text)
- Ensure all rows have same field structure

## Height and Scrolling

Control grid height behavior:

```sail
/* Default: Auto-grow with content (recommended) */
a!gridLayout(
  height: "AUTO"  /* Or omit height parameter */
)

/* Fixed height with scrolling rows */
a!gridLayout(
  height: "MEDIUM"  /* Grid scrolls after reaching height */
)
```

**Height Options:**
- `"AUTO"` (default): Grid grows with content - **USE THIS MOST OF THE TIME**
- `"SHORT"`: ~120px with scrolling
- `"MEDIUM"`: ~240px with scrolling
- `"TALL"`: ~360px with scrolling
- `"EXTRA_TALL"`: ~600px with scrolling

**⚠️ Only set fixed height when:**
- Grid is in a constrained container
- You need predictable vertical space usage
- Multiple components compete for screen space

## Validation Strategies

### Field-Level Validation
Validate individual fields within rows:

```sail
a!textField(
  value: fv!item.email,
  saveInto: fv!item.email,
  labelPosition: "COLLAPSED",
  validations: if(
    and(
      a!isNotNullOrEmpty(fv!item.email),
      or(
        not(contains(fv!item.email, "@")),
        not(contains(fv!item.email, "."))
      )
    ),
    "Please enter a valid email address",
    {}
  )
)
```

### Grid-Level Validation
Validate entire grid data:

```sail
a!gridLayout(
  rows: /* row configuration */,
  validations: if(
    length(local!employees) < 1,
    "At least one employee is required",
    {}
  )
)
```

### Validation Groups
Group related fields for conditional validation:

```sail
a!gridLayout(
  validationGroup: "employeeGrid",
  rows: a!forEach(
    items: local!employees,
    expression: a!gridRowLayout(
      contents: {
        a!textField(
          value: fv!item.firstName,
          saveInto: fv!item.firstName,
          labelPosition: "COLLAPSED",
          required: true,
          requiredMessage: "First name is required",
          validationGroup: "employeeGrid"
        )
      }
    )
  )
)
```

## Complete Examples

### Example 1: Task Management Grid

```sail
a!localVariables(
  local!tasks: {
    a!map(id: 1, taskName: "Review proposals", priority: "high", dueDate: today() + 3, assignedTo: "Sarah J.", status: "in_progress"),
    a!map(id: 2, taskName: "Update documentation", priority: "medium", dueDate: today() + 7, assignedTo: "Mike C.", status: "not_started"),
    a!map(id: 3, taskName: "Client meeting prep", priority: "high", dueDate: today() + 1, assignedTo: "Lisa P.", status: "not_started")
  },
  local!selectedTasks: {},
  local!idCounter: 4,

  a!gridLayout(
    label: "Project Tasks",
    instructions: "Edit tasks directly in the grid. Use 'Add Task' to create new entries.",
    headerCells: {
      a!gridLayoutHeaderCell(label: "Task Name", align: "LEFT"),
      a!gridLayoutHeaderCell(label: "Priority", align: "LEFT"),
      a!gridLayoutHeaderCell(label: "Due Date", align: "LEFT"),
      a!gridLayoutHeaderCell(label: "Assigned To", align: "LEFT"),
      a!gridLayoutHeaderCell(label: "Status", align: "LEFT"),
      a!gridLayoutHeaderCell(label: "Actions", align: "CENTER")
    },
    columnConfigs: {
      a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 3),
      a!gridLayoutColumnConfig(width: "NARROW"),
      a!gridLayoutColumnConfig(width: "NARROW_PLUS"),
      a!gridLayoutColumnConfig(width: "NARROW_PLUS"),
      a!gridLayoutColumnConfig(width: "NARROW_PLUS"),
      a!gridLayoutColumnConfig(width: "NARROW")
    },
    rows: a!forEach(
      items: local!tasks,
      expression: a!gridRowLayout(
        contents: {
          a!textField(
            value: fv!item.taskName,
            saveInto: fv!item.taskName,
            labelPosition: "COLLAPSED",
            required: true,
            disabled: fv!item.status = "completed"
          ),
          a!dropdownField(
            choiceLabels: {"High", "Medium", "Low"},
            choiceValues: {"high", "medium", "low"},
            value: fv!item.priority,
            saveInto: fv!item.priority,
            labelPosition: "COLLAPSED",
            placeholder: "Select"
          ),
          a!dateField(
            value: fv!item.dueDate,
            saveInto: fv!item.dueDate,
            labelPosition: "COLLAPSED",
            validations: if(
              and(
                a!isNotNullOrEmpty(fv!item.dueDate),
                fv!item.dueDate < today()
              ),
              "Due date cannot be in the past",
              {}
            )
          ),
          a!textField(
            value: fv!item.assignedTo,
            saveInto: fv!item.assignedTo,
            labelPosition: "COLLAPSED"
          ),
          a!dropdownField(
            choiceLabels: {"Not Started", "In Progress", "Completed"},
            choiceValues: {"not_started", "in_progress", "completed"},
            value: fv!item.status,
            saveInto: fv!item.status,
            labelPosition: "COLLAPSED"
          ),
          a!buttonArrayLayout(
            buttons: {
              a!buttonWidget(
                label: "Delete",
                icon: "trash",
                style: "OUTLINE",
                size: "SMALL",
                color: "SECONDARY",
                value: fv!index,
                saveInto: a!save(
                  local!tasks,
                  remove(local!tasks, save!value)
                )
              )
            },
            align: "CENTER"
          )
        },
        id: fv!item.id
      )
    ),
    addRowLink: a!dynamicLink(
      label: "Add Task",
      value: a!map(
        id: local!idCounter,
        taskName: "",
        priority: "medium",
        dueDate: today() + 7,
        assignedTo: "",
        status: "not_started"
      ),
      saveInto: {
        a!save(local!tasks, append(local!tasks, save!value)),
        a!save(local!idCounter, local!idCounter + 1)
      }
    ),
    selectable: true,
    selectionValue: local!selectedTasks,
    selectionSaveInto: local!selectedTasks,
    selectionStyle: "CHECKBOX",
    spacing: "STANDARD",
    borderStyle: "STANDARD",
    shadeAlternateRows: true,
    emptyGridMessage: "No tasks available. Click 'Add Task' to create one.",
    rowHeader: 1
  )
)
```

### Example 2: Order Line Items Grid

```sail
a!localVariables(
  local!lineItems: {
    a!map(id: 1, productCode: "WIDGET-001", description: "Standard Widget", quantity: 10, unitPrice: 24.99),
    a!map(id: 2, productCode: "GADGET-205", description: "Premium Gadget", quantity: 5, unitPrice: 149.99)
  },
  local!idCounter: 3,

  a!gridLayout(
    label: "Order Line Items",
    headerCells: {
      a!gridLayoutHeaderCell(label: "Product Code"),
      a!gridLayoutHeaderCell(label: "Description"),
      a!gridLayoutHeaderCell(label: "Quantity", align: "RIGHT"),
      a!gridLayoutHeaderCell(label: "Unit Price", align: "RIGHT"),
      a!gridLayoutHeaderCell(label: "Total", align: "RIGHT"),
      a!gridLayoutHeaderCell(label: "", align: "CENTER")
    },
    columnConfigs: {
      a!gridLayoutColumnConfig(width: "NARROW_PLUS"),
      a!gridLayoutColumnConfig(width: "DISTRIBUTE", weight: 3),
      a!gridLayoutColumnConfig(width: "NARROW"),
      a!gridLayoutColumnConfig(width: "NARROW"),
      a!gridLayoutColumnConfig(width: "NARROW_PLUS"),
      a!gridLayoutColumnConfig(width: "ICON_PLUS")
    },
    rows: a!forEach(
      items: local!lineItems,
      expression: a!gridRowLayout(
        contents: {
          a!textField(
            value: fv!item.productCode,
            saveInto: fv!item.productCode,
            labelPosition: "COLLAPSED",
            required: true
          ),
          a!textField(
            value: fv!item.description,
            saveInto: fv!item.description,
            labelPosition: "COLLAPSED"
          ),
          a!integerField(
            value: fv!item.quantity,
            saveInto: fv!item.quantity,
            labelPosition: "COLLAPSED",
            required: true,
            validations: if(
              or(
                a!isNullOrEmpty(fv!item.quantity),
                fv!item.quantity < 1
              ),
              "Quantity must be at least 1",
              {}
            )
          ),
          a!floatingPointField(
            value: fv!item.unitPrice,
            saveInto: fv!item.unitPrice,
            labelPosition: "COLLAPSED",
            required: true
          ),
          a!richTextDisplayField(
            value: a!richTextItem(
              text: dollar(fv!item.quantity * fv!item.unitPrice),
              style: "STRONG"
            ),
            labelPosition: "COLLAPSED"
          ),
          a!buttonArrayLayout(
            buttons: {
              a!buttonWidget(
                icon: "trash",
                style: "LINK",
                size: "SMALL",
                color: "SECONDARY",
                value: fv!index,
                saveInto: a!save(
                  local!lineItems,
                  remove(local!lineItems, save!value)
                )
              )
            },
            align: "CENTER"
          )
        },
        id: fv!item.id
      )
    ),
    addRowLink: a!dynamicLink(
      label: "Add Line Item",
      value: a!map(
        id: local!idCounter,
        productCode: "",
        description: "",
        quantity: 1,
        unitPrice: 0
      ),
      saveInto: {
        a!save(local!lineItems, append(local!lineItems, save!value)),
        a!save(local!idCounter, local!idCounter + 1)
      }
    ),
    spacing: "DENSE",
    borderStyle: "LIGHT",
    shadeAlternateRows: true,
    emptyGridMessage: "No line items. Click 'Add Line Item' to begin.",
    rowHeader: 2
  )
)
```

## Best Practices

### ✅ DO:
- Always set `labelPosition: "COLLAPSED"` on fields within grid rows
- Use appropriate column width strategy (DISTRIBUTE for most cases)
- Provide clear header labels with `a!gridLayoutHeaderCell()`
- Include validation on required fields
- Use consistent field heights across rows
- Set `rowHeader` parameter to improve screen reader accessibility
- Use `emptyGridMessage` to guide users when grid is empty
- Provide "Add Row" functionality via `addRowLink`
- Use weighted DISTRIBUTE widths for flexible layouts
- Align numeric columns to the RIGHT
- Include unique `id` values for each row in `a!gridRowLayout()`

### ❌ DON'T:
- **NEVER use `a!sideBySideLayout` inside grid rows**
- Put `a!columnsLayout` or `a!cardLayout` in grid cells
- Forget to set `labelPosition: "COLLAPSED"` (creates layout issues)
- Mix "DISTRIBUTE" and "AUTO" width strategies
- Use very wide grids with too many columns (>8 columns gets difficult)
- Set fixed height unless specifically needed for scrolling
- Forget to validate required fields
- Use complex nested components in rows
- Create rows without providing add/delete functionality

### Field Configuration in Rows:
- **Always** set `labelPosition: "COLLAPSED"` on every field
- **Always** set `required: true` and appropriate `validations`
- Use consistent field sizes (`size: "SMALL"` for compact grids)
- Consider using `placeholder` text to guide users
- Use `disabled` parameter conditionally based on row state

### Column Width Guidelines:
- **Short codes/IDs**: `"NARROW"` or `"NARROW_PLUS"`
- **Names**: `"MEDIUM"` or DISTRIBUTE with weight 2-3
- **Descriptions**: DISTRIBUTE with weight 3-5
- **Numbers**: `"NARROW"` with RIGHT alignment
- **Dates**: `"NARROW_PLUS"`
- **Dropdowns**: `"NARROW_PLUS"` to `"MEDIUM"`
- **Action buttons**: `"ICON"` or `"ICON_PLUS"`

## Common Validation Issues

### ❌ WRONG - Using sideBySideLayout in grid row:
```sail
a!gridRowLayout(
  contents: {
    a!sideBySideLayout(    /* ❌ NOT ALLOWED */
      items: {
        a!sideBySideItem(item: a!textField(...)),
        a!sideBySideItem(item: a!dateField(...))
      }
    )
  }
)
```

### ✅ CORRECT - Direct field components:
```sail
a!gridRowLayout(
  contents: {
    a!textField(
      value: fv!item.name,
      saveInto: fv!item.name,
      labelPosition: "COLLAPSED"
    ),
    a!dateField(
      value: fv!item.date,
      saveInto: fv!item.date,
      labelPosition: "COLLAPSED"
    )
  }
)
```

### ❌ WRONG - Missing labelPosition:
```sail
a!textField(
  value: fv!item.name,
  saveInto: fv!item.name
  /* Missing labelPosition causes layout issues */
)
```

### ✅ CORRECT - With labelPosition collapsed:
```sail
a!textField(
  value: fv!item.name,
  saveInto: fv!item.name,
  labelPosition: "COLLAPSED"  /* Required for grid cells */
)
```

## ⚠️ CRITICAL VALIDATION CHECKLIST

### Structure Validation:
- [ ] ✅ Each row contains only field components (no layouts)
- [ ] ✅ All fields have `labelPosition: "COLLAPSED"`
- [ ] ✅ Number of items in `contents` matches number of `columnConfigs`
- [ ] ✅ Number of `headerCells` matches number of `columnConfigs`

### Data Validation:
- [ ] ✅ Required fields have `required: true`
- [ ] ✅ Field validations check for null before comparisons
- [ ] ✅ All rows have unique `id` values in `a!gridRowLayout()`
- [ ] ✅ Data structure uses `a!map()` with consistent fields

### Layout Validation:
- [ ] ✅ Column widths use consistent strategy (all DISTRIBUTE or all fixed)
- [ ] ❌ NO use of "AUTO" width in gridLayout (not supported)
- [ ] ✅ Numeric columns use RIGHT alignment
- [ ] ✅ Action columns use CENTER alignment
- [ ] ✅ `rowHeader` parameter set for accessibility

### Interaction Validation:
- [ ] ✅ `addRowLink` provided for adding rows
- [ ] ✅ Delete buttons or `rowDeletions` configured
- [ ] ✅ Selection configured if batch operations needed
- [ ] ✅ `emptyGridMessage` provides clear guidance
