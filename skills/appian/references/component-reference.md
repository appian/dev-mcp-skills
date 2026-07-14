# SAIL Component Reference

This reference lists commonly used SAIL components with parameters and examples. For a comprehensive list of ALL 146 components, see [registry/components-registry.json](../registry/components-registry.json).

**Related references:**
- For comprehensive component list and instruction file mapping, see [registry/components-registry.json](../registry/components-registry.json)
- For component verification workflow (Tier 2B), see [documentation-lookup-strategy.md](documentation-lookup-strategy.md)
- For functions used in expressions, see [function-reference.md](function-reference.md)

---

## Critical Validation Rule

⚠️ **CRITICAL: Check this list BEFORE using any component**

### Components That DO NOT Exist

These components do not exist in Appian SAIL. Do not use them:

- `a!richTextEditor` - NO rich text editor exists. Use `a!paragraphField` for multi-line text input, `a!styledTextEditorField` for rich text editing with formatting toolbar, or `a!richTextDisplayField` for displaying formatted text (read-only)

### Critical Component Warnings

- **a!formLayout:** Use `titleBar` parameter for the form title, **NOT** `label` (label parameter does not exist)
- **a!sectionLayout:** Does **NOT** have `showBorder` parameter. Use `divider` parameter for visual separation, or wrap in `a!cardLayout(showBorder: true)` if you need borders
- **a!dropdownField:** Use `disabled` parameter for edit mode toggle, **NOT** `readOnly` (readOnly parameter does not exist for dropdowns)
- **a!checkboxField:** Use `disabled` parameter for edit mode toggle, **NOT** `readOnly` (readOnly parameter does not exist for checkboxes)
- **a!gridField:** `showSearchBox`, `showRefreshButton`, and `recordActions` only work with `recordType!` data sources. **DO NOT** use with local data (a!map arrays)

---

## Layout Components

### Top-Level Layouts

| Component | Description | When to Use | Instruction File |
|---|---|---|---|
| `a!formLayout()` | Header, content area, and button footer. Cannot be nested. | Start forms, task forms, any form that collects data | layouts/form-layout-instructions.md |
| `a!headerContentLayout()` | Content beneath a card or billboard header, flush with page edge. | Dashboards, landing pages, summary views | layouts/header-content-layout-instructions.md |
| `a!paneLayout()` | Two or three vertical panes with independent scrolling | Multi-panel interfaces, side-by-side views | layouts/pane-layout-instructions.md |
| `a!wizardLayout()` | Multi-step form with step indicator. Uses `a!wizardStepLayout()` children. | Complex forms organized into sequential steps | layouts/wizard-layout-instructions.md |

### a!formLayout

| Parameter | Type | Description |
|---|---|---|
| `titleBar` | Text | Title displayed at the top of the form. **Use this, NOT `label`** |
| `contents` | Any Type | Components and layouts displayed in the form body |
| `buttons` | Button Layout | `a!buttonLayout()` with primary and secondary buttons |
| `validations` | List of Text | Form-level validation messages displayed at the top |
| `validationGroup` | Text | Validation group name for targeted validation |
| `showWhen` | Boolean | Controls visibility. Default: true |
| `focusOnFirstInput` | Boolean | Auto-focus the first input field on load |
| `isButtonFooterFixed` | Boolean | Fix buttons to the bottom of the viewport |

### a!headerContentLayout

| Parameter | Type | Description |
|---|---|---|
| `header` | Any Type | Card or billboard layout displayed as the header |
| `contents` | Any Type | Components and layouts displayed below the header |
| `showWhen` | Boolean | Controls visibility. Default: true |
| `backgroundColor` | Text | Background color. Valid: `"WHITE"`, `"TRANSPARENT"` |

### Container Layouts

| Component | Description | Key Parameters |
|---|---|---|
| `a!sectionLayout()` | Groups content under a heading | `label`, `contents`, `showWhen`, `isCollapsible`, `isInitiallyCollapsed`, `validations` |
| `a!columnsLayout()` | Side-by-side columns | `columns` (list of `a!columnLayout()`), `alignVertical`, `stackWhen`, `showDividers`, `spacing` |
| `a!columnLayout()` | Single column inside `a!columnsLayout()` | `contents`, `width`, `showWhen` |
| `a!cardLayout()` | Bordered/shaded container | `contents`, `style`, `showBorder`, `padding`, `showWhen`, `link`, `tooltip` |
| `a!boxLayout()` | Titled container with header bar | `label`, `contents`, `style`, `showWhen`, `isCollapsible`, `padding` |
| `a!sideBySideLayout()` | Inline items side by side | `items` (list of `a!sideBySideItem()`), `alignVertical`, `showWhen`, `spacing` — See layouts/sidebyside-layout-instructions.md |
| `a!sideBySideItem()` | Single item inside `a!sideBySideLayout()` | `item`, `width`, `showWhen` |
| `a!tabLayout()` | Content organized into selectable tabs | `tabs` (list of `a!tabItem()`), `value`, `saveInto`, `showWhen` — See layouts/tab-layout-instructions.md |

### a!columnsLayout Details

| Parameter | Type | Description |
|---|---|---|
| `columns` | List | `a!columnLayout()` items |
| `alignVertical` | Text | `"TOP"` (default), `"MIDDLE"`, `"BOTTOM"` |
| `stackWhen` | List of Text | `"PHONE"` (default), `"TABLET_PORTRAIT"`, `"TABLET_LANDSCAPE"`, `"DESKTOP"`, `"DESKTOP_WIDE"`, `"NEVER"` |
| `showDividers` | Boolean | Show vertical dividers between columns. Default: false |
| `spacing` | Text | `"STANDARD"` (default), `"NONE"`, `"DENSE"`, `"SPARSE"` |
| `marginAbove` | Text | `"NONE"` (default), `"EVEN_LESS"`, `"LESS"`, `"STANDARD"`, `"MORE"`, `"EVEN_MORE"` |
| `marginBelow` | Text | `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"` |

### a!columnLayout Details

| Parameter | Type | Description |
|---|---|---|
| `contents` | Any Type | Components and layouts in this column |
| `width` | Text | `"AUTO"` (default), `"EXTRA_NARROW"`, `"NARROW"`, `"NARROW_PLUS"`, `"MEDIUM"`, `"MEDIUM_PLUS"`, `"WIDE"`, `"WIDE_PLUS"`, `"1X"` through `"10X"` (relative) |
| `showWhen` | Boolean | Controls visibility. Default: true |

### a!cardLayout Details

| Parameter | Type | Description |
|---|---|---|
| `contents` | Any Type | Components and layouts inside the card |
| `style` | Text | `"NONE"` (default), `"STANDARD"`, `"ACCENT"`, `"SUCCESS"`, `"INFO"`, `"WARN"`, `"ERROR"` |
| `showBorder` | Boolean | Show card border. Default: true |
| `padding` | Text | `"LESS"` (default), `"NONE"`, `"EVEN_LESS"`, `"STANDARD"`, `"MORE"`, `"EVEN_MORE"` |
| `link` | Link | Makes the entire card clickable |
| `showWhen` | Boolean | Controls visibility. Default: true |
| `height` | Text | `"AUTO"` (default), `"EXTRA_SHORT"`, `"SHORT"`, `"SHORT_PLUS"`, `"MEDIUM"`, `"MEDIUM_PLUS"`, `"TALL"`, `"TALL_PLUS"`, `"EXTRA_TALL"` |

### a!sectionLayout Details

| Parameter | Type | Description |
|---|---|---|
| `label` | Text | Section heading |
| `contents` | Any Type | Components and layouts in this section |
| `showWhen` | Boolean | Controls visibility. Default: true |
| `isCollapsible` | Boolean | Allow section to collapse. Default: false |
| `isInitiallyCollapsed` | Boolean | Start collapsed. Default: false |
| `validations` | List of Validation | Section-level validation messages |
| `divider` | Text | `"NONE"` (default), `"BELOW"`, `"ABOVE"` — adds line divider above/below section |
| `dividerColor` | Text | `"STANDARD"`, `"SECONDARY"`, `"ACCENT"` — divider line color |
| `dividerWeight` | Text | `"THIN"`, `"MEDIUM"`, `"THICK"` — divider line thickness |
| `marginAbove` | Text | `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"` |
| `marginBelow` | Text | `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"` |
| `labelHeadingTag` | Text | `"H1"`, `"H2"`, `"H3"`, `"H4"`, `"H5"`, `"H6"` — required for accessibility when `isCollapsible: true` |

**Note:** `a!sectionLayout` does **NOT** have a `showBorder` parameter. To add borders around a section, wrap it in `a!cardLayout(showBorder: true)`.

### a!paneLayout Details

| Parameter | Type | Description |
|---|---|---|
| `panes` | List | List of `a!paneComponent()` items |
| `paneSplit` | Text | Split ratio: `"1-1"` (equal), `"1-2"`, `"2-1"`, `"1-3"`, `"3-1"`, `"1-1-1"` (three panes) |
| `showWhen` | Boolean | Controls visibility. Default: true |

**See layouts/pane-layout-instructions.md for detailed guidance**

### a!sideBySideLayout Details

| Parameter | Type | Description |
|---|---|---|
| `items` | List | List of `a!sideBySideItem()` items |
| `alignVertical` | Text | `"TOP"` (default), `"MIDDLE"`, `"BOTTOM"` |
| `spacing` | Text | `"STANDARD"` (default), `"NONE"`, `"DENSE"`, `"SPARSE"` |
| `showWhen` | Boolean | Controls visibility. Default: true |
| `marginAbove` | Text | `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"` |
| `marginBelow` | Text | `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"` |

**See layouts/sidebyside-layout-instructions.md for detailed guidance**

### a!tabLayout Details

| Parameter | Type | Description |
|---|---|---|
| `tabs` | List | List of `a!tabItem()` items |
| `value` | Integer | Currently selected tab index (1-based) |
| `saveInto` | Save | Target for saving selected tab index |
| `showWhen` | Boolean | Controls visibility. Default: true |
| `marginAbove` | Text | `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"` |
| `marginBelow` | Text | `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"` |

**See layouts/tab-layout-instructions.md for detailed guidance**

## Input Components

### Text and Number Inputs

| Component | Description | Key Parameters |
|---|---|---|
| `a!textField()` | Single-line text input | `label`, `value`, `saveInto`, `required`, `requiredMessage`, `readOnly`, `disabled`, `placeholder`, `validations`, `showWhen`, `characterLimit`, `inputPurpose` |
| `a!paragraphField()` | Multi-line text input | Same as textField plus `height` |
| `a!integerField()` | Integer number input | `label`, `value`, `saveInto`, `required`, `readOnly`, `validations`, `showWhen` |
| `a!floatingPointField()` | Decimal number input | Same as integerField |
| `a!dateField()` | Date picker | `label`, `value`, `saveInto`, `required`, `readOnly`, `validations`, `showWhen` |
| `a!dateTimeField()` | Date and time picker | Same as dateField |
| `a!timeDisplayField()` | Time display field | Same as dateField |

### Selection Inputs

| Component | Description | Key Parameters |
|---|---|---|
| `a!dropdownField()` | Single-select dropdown | `label`, `placeholder`, `choiceLabels`, `choiceValues`, `data`, `sort`, `value`, `saveInto`, `required`, `showWhen`, `searchDisplay` |
| `a!multipleDropdownField()` | Multi-select dropdown | Same as dropdownField but `value` is a list |
| `a!radioButtonField()` | Radio button group | `label`, `choiceLabels`, `choiceValues`, `data`, `sort`, `value`, `saveInto`, `required`, `choiceLayout`, `showWhen` |
| `a!checkboxField()` | Checkbox group | `label`, `choiceLabels`, `choiceValues`, `data`, `sort`, `value`, `saveInto`, `required`, `choiceLayout`, `showWhen` |
| `a!cardChoiceField()` | Card-based selection (single or multiple) | `label`, `data`, `cardTemplate`, `value`, `saveInto`, `maxSelections`, `showWhen` — See components/card-choice-field-instructions.md |
| `a!pickerField()` | Type-ahead picker | `label`, `labelPosition`, `value`, `saveInto`, `maxSelections`, `suggestFunction`, `required`, `showWhen` |

### a!dropdownField Details

| Parameter | Type | Description |
|---|---|---|
| `label` | Text | Field label |
| `labelPosition` | Text | `"ABOVE"` (default), `"ADJACENT"`, `"COLLAPSED"`, `"JUSTIFIED"` |
| `placeholder` | Text | Placeholder text when no value selected |
| `choiceLabels` | List | Display labels for each option. **Always required, including when `data` is set.** With `data`, use a record field reference (e.g., `recordType!Priority.fields.label`); use `fv!data[...]` only to format/combine multiple fields into one label |
| `choiceValues` | List | Values corresponding to each label. **Always required, including when `data` is set.** With `data`, use a record field reference — usually the primary key (e.g., `recordType!Priority.fields.id`) |
| `value` | Any Type | Currently selected value |
| `saveInto` | Save | Target for saving selected value |
| `required` | Boolean | Whether selection is required. Default: false |
| `requiredMessage` | Text | Custom required message |
| `disabled` | Boolean | Disable the dropdown. Default: false |
| `validations` | List of Text | Validation messages |
| `showWhen` | Boolean | Controls visibility. Default: true |
| `searchDisplay` | Text | `"AUTO"` (default), `"ON"`, `"OFF"` — controls search within dropdown |
| `data` | Record Type / `a!recordData()` | Records-powered data source. Replaces querying into a local variable — but does **NOT** replace `choiceValues`/`choiceLabels`, which are still required and point at record fields. See `references/dropdown-patterns.md` |
| `sort` | List of Sort Info | Sort order for records-powered choices (used with `data`). Use `"choiceLabels"` or `"choiceValues"` as the `a!sortInfo()` field |
| `instructions` | Text | Help text displayed below the label |
| `helpTooltip` | Text | Tooltip shown on hover |
| `accessibilityText` | Text | Screen reader text |
| `validationGroup` | Text | Group name for targeted validation |
| `marginAbove` | Text | `"NONE"` (default), `"EVEN_LESS"`, `"LESS"`, `"STANDARD"`, `"MORE"`, `"EVEN_MORE"` |
| `marginBelow` | Text | `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"` |

### a!radioButtonField / a!checkboxField Details

Both take the same choice parameters. `a!checkboxField` and `a!multipleDropdownField` hold a **list** in `value`; `a!radioButtonField` and `a!dropdownField` hold a single value.

| Parameter | Type | Description |
|---|---|---|
| `choiceLabels` | List | Display labels for each option. **Always required, including when `data` is set** (use a record field reference, or `fv!data[...]` to combine fields) |
| `choiceValues` | List | Values corresponding to each label. **Always required, including when `data` is set** (use a record field reference — usually the primary key) |
| `data` | Record Type / `a!recordData()` | Records-powered data source. Does **NOT** replace `choiceValues`/`choiceLabels`. Supported on `a!dropdownField`, `a!multipleDropdownField`, `a!checkboxField`, `a!radioButtonField` |
| `sort` | List of Sort Info | Sort order for records-powered choices. Use `"choiceLabels"` or `"choiceValues"` as the `a!sortInfo()` field |
| `choiceLayout` | Text | `"HORIZONTAL"` (default), `"STACKED"` |
| `value` | Any Type | Currently selected value(s) |
| `saveInto` | Save | Target for saving selected value |

### Other Input Components

| Component | Description |
|---|---|
| `a!fileUploadField()` | File upload with `value`, `saveInto`, `maxSelections`, `validations` |
| `a!imageUploadField()` | Image upload |
| `a!encryptedTextField()` | Masked text input for sensitive data |
| `a!richTextEditor()` | Rich text editor with formatting toolbar |

### Common Input Parameters

These parameters are shared across most input components:

| Parameter | Type | Description |
|---|---|---|
| `label` | Text | Field label displayed to the user |
| `labelPosition` | Text | `"ABOVE"` (default), `"ADJACENT"`, `"COLLAPSED"`, `"JUSTIFIED"` |
| `value` | Any Type | Current field value |
| `saveInto` | Save | Where to save the value on change |
| `required` | Boolean | Whether the field is required. Default: false |
| `requiredMessage` | Text | Custom message when required field is empty |
| `readOnly` | Boolean | Display value without allowing edits. Default: false |
| `disabled` | Boolean | Show field but prevent interaction. Default: false |
| `placeholder` | Text | Placeholder text when field is empty |
| `validations` | List of Text | Validation error messages |
| `showWhen` | Boolean | Controls visibility. Default: true |
| `helpTooltip` | Text | Help text shown on hover |
| `accessibilityText` | Text | Screen reader text |
| `validationGroup` | Text | Group name for targeted validation |

## Display Components

| Component | Description | Key Parameters |
|---|---|---|
| `a!richTextDisplayField()` | Rich text with formatting, links, icons | `value` (list of `a!richTextItem()`), `showWhen` |
| `a!richTextItem()` | Text element inside rich text | `text`, `style`, `size`, `color`, `link`, `showWhen` |
| `a!richTextIcon()` | Icon inside rich text | `icon`, `color`, `size`, `link`, `altText` |
| `a!richTextImage()` | Image inside rich text | `image`, `link`, `altText` |
| `a!richTextBulletedList()` | Bulleted list in rich text | `items` (list of `a!richTextItem()`), `showWhen` |
| `a!richTextNumberedList()` | Numbered list in rich text | `items` (list of `a!richTextItem()`), `showWhen` |
| `a!richTextListItem()` | Item in a list | `text`, `link`, `showWhen` |
| `a!stampField()` | Large icon with label for KPIs | `icon`, `text`, `backgroundColor`, `contentColor`, `size`, `showWhen` |
| `a!tagField()` | Colored tag/badge | `tags` (list of `a!tagItem()`), `size`, `showWhen` |
| `a!tagItem()` | Single tag inside tag field | `text`, `backgroundColor`, `textColor` |
| `a!imageField()` | Image display | `images`, `size`, `showWhen`, `isThumbnail` — See components/image-field-instructions.md |
| `a!documentImage()` | Image from Appian document | `document`, `altText`, `link` |
| `a!webImage()` | Image from URL | `source`, `altText`, `link` |
| `a!progressBarField()` | Progress bar | `percentage`, `label`, `style`, `showWhen` |
| `a!gaugeField()` | Circular gauge | `percentage`, `primaryText`, `secondaryText`, `style`, `showWhen` |
| `a!milestoneField()` | Step indicator | `steps`, `activeStep`, `showWhen` |
| `a!linkField()` | Standalone link display | `links`, `showWhen` |

### a!richTextDisplayField Details

```
a!richTextDisplayField(
  value: {
    a!richTextItem(text: "Total: ", style: "STRONG"),
    a!richTextItem(text: dollar(local!total), size: "LARGE", color: "ACCENT"),
    " ",
    a!richTextIcon(icon: "check-circle", color: "POSITIVE", altText: "Complete")
  }
)
```

Rich text style values: `"STRONG"`, `"EMPHASIS"`, `"UNDERLINE"`, `"STRIKETHROUGH"`
Rich text size values: `"SMALL"`, `"MEDIUM"` (default), `"MEDIUM_PLUS"`, `"LARGE"`, `"LARGE_PLUS"`, `"EXTRA_LARGE"`
Rich text color values: `"STANDARD"`, `"ACCENT"`, `"POSITIVE"`, `"NEGATIVE"`, `"SECONDARY"`

### a!stampField Details

| Parameter | Type | Description |
|---|---|---|
| `icon` | Text | Icon name (e.g., `"briefcase"`, `"users"`, `"chart-bar"`) |
| `text` | Text | Label text below the icon |
| `backgroundColor` | Text | `"ACCENT"`, `"POSITIVE"`, `"NEGATIVE"`, `"SECONDARY"`, or hex color |
| `contentColor` | Text | `"STANDARD"` (default), `"ACCENT"`, `"POSITIVE"`, `"NEGATIVE"`, `"SECONDARY"` |
| `size` | Text | `"SMALL"`, `"MEDIUM"` (default), `"LARGE"`, `"TINY"` |
| `showWhen` | Boolean | Controls visibility |

## Grid Components

### a!gridField (Grid Field)

⚠️ **CRITICAL:** `showSearchBox`, `showRefreshButton`, and `recordActions` only work with `recordType!` data sources. **DO NOT** use with local data (a!map arrays).

**See components/grid-field-instructions.md for detailed guidance**

| Parameter | Type | Description |
|---|---|---|
| `label` | Text | Grid title |
| `data` | Record Data | `a!recordData()` source |
| `columns` | List | `a!gridColumn()` definitions |
| `pageSize` | Integer | Rows per page. Default: 10 |
| `showSearchBox` | Boolean | Show search input. Default: false **⚠️ recordType! data only** |
| `showRefreshButton` | Boolean | Show refresh button. Default: false **⚠️ recordType! data only** |
| `showExportButton` | Boolean | Show export to Excel button. Default: false |
| `selectable` | Boolean | Allow row selection. Default: false |
| `selectionStyle` | Text | `"CHECKBOX"` (default), `"ROW_HIGHLIGHT"` |
| `selectionValue` | List | Currently selected rows |
| `selectionSaveInto` | Save | Where to save selections |
| `showWhen` | Boolean | Controls visibility |
| `emptyGridMessage` | Text | Message when no data |
| `rowHeader` | Integer | Column index to use as row header for accessibility |

### a!gridColumn

| Parameter | Type | Description |
|---|---|---|
| `label` | Text | Column header |
| `value` | Any Type | Cell value expression using `fv!row` |
| `sortField` | Record Field | Field to sort by when column header is clicked |
| `align` | Text | `"START"` (default), `"CENTER"`, `"END"` |
| `width` | Text | `"AUTO"` (default), `"ICON"`, `"ICON_PLUS"`, `"NARROW"`, `"NARROW_PLUS"`, `"MEDIUM"`, `"MEDIUM_PLUS"`, `"WIDE"`, `"WIDE_PLUS"`, `"1X"` through `"10X"` |
| `showWhen` | Boolean | Controls visibility |

### a!recordData

| Parameter | Type | Description |
|---|---|---|
| `recordType` | Record Type | The record type to query |
| `filters` | Query Filter | `a!queryFilter()` or `a!queryLogicalExpression()` |
| `fields` | List | Specific fields to include (optional — all fields returned by default) |

### a!gridLayout (Editable Grid)

**See components/grid-layout-instructions.md for detailed guidance**

| Parameter | Type | Description |
|---|---|---|
| `label` | Text | Grid title |
| `headerCells` | List | `a!gridLayoutHeaderCell()` definitions |
| `rows` | List | `a!gridRowLayout()` items (typically from `a!forEach()`) |
| `addRowLink` | Link | `a!dynamicLink()` to add a new row |
| `totalCount` | Integer | Total row count for paging |
| `showWhen` | Boolean | Controls visibility |
| `validations` | List of Text | Grid-level validation messages |

### a!gridRowLayout

| Parameter | Type | Description |
|---|---|---|
| `contents` | List | Cell components (one per header cell) |
| `id` | Any Type | Unique row identifier |
| `showWhen` | Boolean | Controls visibility |

## Charts

**See components/chart-instructions.md for detailed guidance on all chart types**

| Component | Description | Key Parameters |
|---|---|---|
| `a!areaChartField()` | Line chart with shaded area for trend visualization | `label`, `categories`, `series` (list of `a!chartSeries()`), `yAxisMax`, `yAxisMin`, `showWhen` |
| `a!barChartField()` | Horizontal bar chart for value comparison | `label`, `categories`, `series` (list of `a!chartSeries()`), `xAxisMax`, `xAxisMin`, `stacking`, `showWhen` |
| `a!columnChartField()` | Vertical bar chart for time-based data | `label`, `categories`, `series` (list of `a!chartSeries()`), `yAxisMax`, `yAxisMin`, `stacking`, `showWhen` |
| `a!lineChartField()` | Line chart for trend visualization | `label`, `categories`, `series` (list of `a!chartSeries()`), `yAxisMax`, `yAxisMin`, `showWhen` |
| `a!pieChartField()` | Circular slices showing parts of whole | `label`, `series` (list of `a!chartSeries()`), `showDataLabels`, `showAsPercentage`, `showWhen` |
| `a!scatterChartField()` | Two-dimensional relationship visualization | `label`, `series` (list of `a!chartSeries()`), `xAxisTitle`, `yAxisTitle`, `showWhen` |

**Note:** All charts use `a!chartSeries()` to define data. Common patterns include:
- Record-backed charts: Use `data` parameter with `a!recordData()`
- Local data charts: Use arrays with `categories` and `series` parameters
- Styling: Use `a!chartCustomColorScheme()` for custom colors
- Reference lines: Use `a!chartReferenceLine()` for thresholds

## Action Components

### a!buttonWidget

**See components/button-instructions.md for detailed guidance**

| Parameter | Type | Description |
|---|---|---|
| `label` | Text | Button text |
| `style` | Text | `"SOLID"`, `"OUTLINE"` (default), `"GHOST"`, `"LINK"` — **only these 4 are valid**; `"PRIMARY"`, `"SECONDARY"`, `"NORMAL"` do not exist |
| `color` | Text | `"ACCENT"`, `"SECONDARY"`, `"NEGATIVE"`, or hex `#RRGGBB` — use `"ACCENT"` for primary actions, `"NEGATIVE"` for destructive |
| `submit` | Boolean | Submit the form on click. Default: false |
| `validate` | Boolean | Trigger validation before submit. Default: true |
| `validationGroup` | Text | Only validate fields in this group |
| `value` | Any Type | Value to save |
| `saveInto` | Save | Where to save the value |
| `disabled` | Boolean | Disable the button. Default: false |
| `showWhen` | Boolean | Controls visibility |
| `confirmHeader` | Text | Confirmation dialog header |
| `confirmMessage` | Text | Confirmation dialog message |
| `confirmButtonLabel` | Text | Confirmation dialog confirm button text |
| `icon` | Text | Icon name to display on the button |
| `size` | Text | `"STANDARD"` (default), `"SMALL"` |

### a!buttonLayout

| Parameter | Type | Description |
|---|---|---|
| `primaryButtons` | List | Primary action buttons (right-aligned) |
| `secondaryButtons` | List | Secondary action buttons (left-aligned) |
| `showWhen` | Boolean | Controls visibility |

### a!buttonArrayLayout

| Parameter | Type | Description |
|---|---|---|
| `buttons` | List | `a!buttonWidget()` items |
| `align` | Text | `"START"`, `"CENTER"`, `"END"` (default) |
| `showWhen` | Boolean | Controls visibility |

### Link Types

| Component | Description | Key Parameters |
|---|---|---|
| `a!recordLink()` | Navigate to a record view | `label`, `recordType`, `identifier` |
| `a!dynamicLink()` | In-page action (no navigation) | `label`, `saveInto`, `showWhen` |
| `a!startProcessLink()` | Start a process model | `label`, `processModel`, `processParameters`, `showWhen` |
| `a!safeLink()` | Open external URL | `label`, `uri`, `showWhen` |
| `a!documentDownloadLink()` | Download a document | `label`, `document`, `showWhen` |
| `a!userRecordLink()` | Link to a user record | `label`, `user`, `showWhen` |
| `a!reportLink()` | Link to a report | `label`, `report`, `showWhen` |
| `a!newsEntryLink()` | Link to a news entry | `label`, `newsEntry`, `showWhen` |

## Record Action Components

Use these to surface configured record actions (LIST_ACTION / RELATED_ACTION) on an interface. Configuring an action on the record type does **not** make it appear — related actions shown outside a record view (e.g., grid rows), and list actions on custom grids, must be placed explicitly. See `references/record-types.md` for configuration and `references/appian-workflow-patterns.md` ("Surfacing Record Actions on Interfaces") for when to use each placement.

### a!recordActionField

| Parameter | Type | Description |
|---|---|---|
| `actions` | Any Type | List of record action items to display, configured with `a!recordActionItem()` |
| `style` | Text | How the actions are displayed. Valid: `"TOOLBAR"` (default), `"LINKS"`, `"CARDS"`, `"SIDEBAR"`, `"CALL_TO_ACTION"`, `"MENU"`, `"MENU_ICON"`, `"TOOLBAR_PRIMARY"`, `"SIDEBAR_PRIMARY"` |
| `display` | Text | How each action label is displayed. Valid: `"LABEL_AND_ICON"` (default), `"LABEL"`, `"ICON"` |
| `openActionsIn` | Text | How actions open. Valid: `"DIALOG"` (default), `"NEW_TAB"`, `"SAME_TAB"`. Does not apply to Appian Mobile |
| `align` | Text | Alignment of the action(s). Valid: `"START"`, `"CENTER"`, `"END"` |
| `accessibilityText` | Text | Additional text announced by screen readers; no visible change |
| `showWhen` | Boolean | Controls visibility. When `false`, hidden and not evaluated. Default: `true` |
| `securityOnDemand` | Boolean | When security is evaluated for `"MENU"`/`"MENU_ICON"` styles. Default `true` = on click; `false` = on interface load |

### a!recordActionItem

| Parameter | Type | Description |
|---|---|---|
| `action` | Record Action Reference | A record action reference via the `recordType!` domain, e.g., `recordType!{uuid}Case.actions.editCase`. Uses the display name, process model, icon, context, and visibility set in the record type's action config |
| `identifier` | Any Type | The record's identifier (typically the primary key value) — **required when `action` references a related action**. For composite primary keys, provide all key field values. Not needed for a record list action |

> On a custom `a!gridField()`, list actions and related actions are surfaced via the grid's `recordActions` parameter (which also takes `a!recordActionItem()` items) — see `references/sail.md`.

## Wizard Components

### a!wizardLayout

| Parameter | Type | Description |
|---|---|---|
| `steps` | List | `a!wizardStepLayout()` items |
| `buttons` | Button Layout | `a!buttonLayout()` for navigation buttons |
| `validations` | List of Text | Wizard-level validation messages |
| `showWhen` | Boolean | Controls visibility |

Inside wizard step expressions, use:
- `fv!activeStepIndex` — the currently active step (1-based)
- `fv!isFirstStep` — Boolean, true on the first step
- `fv!isLastStep` — Boolean, true on the last step

### a!wizardStepLayout

| Parameter | Type | Description |
|---|---|---|
| `label` | Text | Step label shown in the step indicator |
| `contents` | Any Type | Components and layouts for this step |
| `showWhen` | Boolean | Controls visibility |
| `validations` | List of Text | Step-level validation messages |

## Common Patterns

### Label-Value Display (Summary View)

```
a!sideBySideLayout(
  items: {
    a!sideBySideItem(
      item: a!richTextDisplayField(
        value: a!richTextItem(text: "Status:", style: "STRONG")
      ),
      width: "MINIMIZE"
    ),
    a!sideBySideItem(
      item: a!richTextDisplayField(
        value: a!richTextItem(text: local!record[recordType!Case.fields.status])
      )
    )
  }
)
```

### KPI Card

```
a!cardLayout(
  contents: {
    a!stampField(
      icon: "briefcase",
      text: "Open Cases",
      backgroundColor: "ACCENT",
      size: "MEDIUM"
    ),
    a!richTextDisplayField(
      value: a!richTextItem(
        text: tostring(local!openCount),
        size: "EXTRA_LARGE",
        style: "STRONG"
      )
    )
  },
  padding: "STANDARD"
)
```

### Confirmation Dialog on Button

```
a!buttonWidget(
  label: "Delete",
  style: "SOLID",
  color: "NEGATIVE",
  confirmHeader: "Confirm Delete",
  confirmMessage: "Are you sure you want to delete this record? This action cannot be undone.",
  confirmButtonLabel: "Delete",
  value: true,
  saveInto: local!confirmDelete,
  submit: true
)
```
