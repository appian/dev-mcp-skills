# SAIL Component Reference

## Layout Components

### Top-Level Layouts

| Component | Description | When to Use |
|---|---|---|
| `a!formLayout()` | Header, content area, and button footer. Cannot be nested. | Start forms, task forms, any form that collects data |
| `a!headerContentLayout()` | Content beneath a card or billboard header, flush with page edge. | Dashboards, landing pages, summary views |
| `a!wizardLayout()` | Multi-step form with step indicator. Uses `a!wizardStepLayout()` children. | Complex forms organized into sequential steps |

### a!formLayout

| Parameter | Type | Description |
|---|---|---|
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
| `a!sideBySideLayout()` | Inline items side by side | `items` (list of `a!sideBySideItem()`), `alignVertical`, `showWhen`, `spacing` |
| `a!sideBySideItem()` | Single item inside `a!sideBySideLayout()` | `item`, `width`, `showWhen` |

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
| `divider` | Text | `"NONE"` (default), `"BELOW"`, `"ABOVE"` |

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
| `a!timeField()` | Time picker | Same as dateField |

### Selection Inputs

| Component | Description | Key Parameters |
|---|---|---|
| `a!dropdownField()` | Single-select dropdown | `label`, `placeholder`, `choiceLabels`, `choiceValues`, `data`, `value`, `saveInto`, `required`, `showWhen`, `searchDisplay` |
| `a!multipleDropdownField()` | Multi-select dropdown | Same as dropdownField but `value` is a list |
| `a!radioButtonField()` | Radio button group | `label`, `choiceLabels`, `choiceValues`, `value`, `saveInto`, `required`, `choiceLayout`, `showWhen` |
| `a!checkboxField()` | Checkbox group | `label`, `choiceLabels`, `choiceValues`, `value`, `saveInto`, `required`, `choiceLayout`, `showWhen` |
| `a!pickerField()` | Type-ahead picker | `label`, `labelPosition`, `value`, `saveInto`, `maxSelections`, `suggestFunction`, `required`, `showWhen` |

### a!dropdownField Details

| Parameter | Type | Description |
|---|---|---|
| `label` | Text | Field label |
| `labelPosition` | Text | `"ABOVE"` (default), `"ADJACENT"`, `"COLLAPSED"`, `"JUSTIFIED"` |
| `placeholder` | Text | Placeholder text when no value selected |
| `choiceLabels` | List of Text | Display labels for each option |
| `choiceValues` | List | Values corresponding to each label |
| `value` | Any Type | Currently selected value |
| `saveInto` | Save | Target for saving selected value |
| `required` | Boolean | Whether selection is required. Default: false |
| `requiredMessage` | Text | Custom required message |
| `disabled` | Boolean | Disable the dropdown. Default: false |
| `validations` | List of Text | Validation messages |
| `showWhen` | Boolean | Controls visibility. Default: true |
| `searchDisplay` | Text | `"AUTO"` (default), `"ON"`, `"OFF"` — controls search within dropdown |
| `data` | Record Type | Records-powered data source — use instead of `choiceLabels`/`choiceValues` when choices come from a record type |
| `sort` | Sort Info | Sort order for records-powered choices (used with `data`) |
| `instructions` | Text | Help text displayed below the label |
| `helpTooltip` | Text | Tooltip shown on hover |
| `accessibilityText` | Text | Screen reader text |
| `validationGroup` | Text | Group name for targeted validation |
| `marginAbove` | Text | `"NONE"` (default), `"EVEN_LESS"`, `"LESS"`, `"STANDARD"`, `"MORE"`, `"EVEN_MORE"` |
| `marginBelow` | Text | `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"` |

### a!radioButtonField Details

| Parameter | Type | Description |
|---|---|---|
| `choiceLabels` | List of Text | Display labels for each option |
| `choiceValues` | List | Values corresponding to each label |
| `choiceLayout` | Text | `"HORIZONTAL"` (default), `"STACKED"` |
| `value` | Any Type | Currently selected value |
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
| `a!stampField()` | Large icon with label for KPIs | `icon`, `text`, `backgroundColor`, `contentColor`, `size`, `showWhen` |
| `a!tagField()` | Colored tag/badge | `tags` (list of `a!tagItem()`), `size`, `showWhen` |
| `a!tagItem()` | Single tag inside tag field | `text`, `backgroundColor`, `textColor` |
| `a!imageField()` | Image display | `images`, `size`, `showWhen`, `isThumbnail` |
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

### a!gridField (Record-Backed Grid)

| Parameter | Type | Description |
|---|---|---|
| `label` | Text | Grid title |
| `data` | Record Data | `a!recordData()` source |
| `columns` | List | `a!gridColumn()` definitions |
| `pageSize` | Integer | Rows per page. Default: 10 |
| `showSearchBox` | Boolean | Show search input. Default: false |
| `showRefreshButton` | Boolean | Show refresh button. Default: false |
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

## Action Components

### a!buttonWidget

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
