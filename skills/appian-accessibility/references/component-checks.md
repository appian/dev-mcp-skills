# Component-to-Rule Quick Reference

Use this reference to quickly identify which accessibility rules apply to a given SAIL component.

## Input Components (all types)

Applies to: `a!textField`, `a!paragraphField`, `a!integerField`, `a!decimalField`, `a!dateField`, `a!timeField`, `a!dropdownField`, `a!multipleDropdownField`, `a!pickerFieldCustom`, `a!pickerFieldUsers`, `a!pickerFieldGroups`, `a!pickerFieldRecords`, `a!encryptedTextField`

| Rule ID | What to Check |
|---|---|
| INPUT-LABEL | `label` is not null |
| INPUT-LABEL-MATCH | If `labelPosition: "COLLAPSED"`, `label` still has a value |
| INPUT-DUPLICATES | If same `label` appears multiple times, `accessibilityText` provides context |
| REQUIRED-PARAM | If the field is required, `required: true` is set |
| VALIDATION-OOTB | Errors use `validations`/`validationMessage` parameters |
| VALIDATION-INCLUDES-NAME | Validation/error message text includes the input's `label` value |
| INPUT-INSTRUCTIONS | Format/range guidance uses the `instructions` parameter |

## a!textField (additional)

| Rule ID | What to Check |
|---|---|
| INPUT-PURPOSE | If personal info (name, email, phone, address), `inputPurpose` is set |

## a!checkboxField, a!radioButtonField

| Rule ID | What to Check |
|---|---|
| CHECKBOX-CHOICE-LABELS | `choiceLabels` has no null values |
| CHECKBOX-GROUP-LABEL | `label` is set when more than one option exists |

## a!toggleField

| Rule ID | What to Check |
|---|---|
| TOGGLE-LABEL | `choiceLabel` is not null |

## a!gridField

| Rule ID | What to Check |
|---|---|
| GRID-LABEL | `label` is set and describes the grid content |
| GRID-ROW-HEADER | `rowHeader` is set to the appropriate column position |
| GRID-INSTRUCTIONS | Instructions use the `instructions` parameter; text says "table" not "grid" |
| GRID-SELECTION-A11Y | If controls enable on row selection, `accessibilityText` warns users; text says "table" not "grid" |

## a!gridColumn

| Rule ID | What to Check |
|---|---|
| GRID-COLUMN-HEADER | `label` is not null for data columns |
| GRID-EMPTY-COLUMNS | Column has both valid `label` and `value` — no empty spacer columns |

## a!headingField

| Rule ID | What to Check |
|---|---|
| HEADING-SEMANTIC | Used instead of styled rich text for headings |
| HEADING-LEVEL | `headingTag` is set to appropriate level (H1-H6) |

## a!sectionLayout

| Rule ID | What to Check |
|---|---|
| SECTION-LABEL | If expandable (`isCollapsible: true`), `label` is set |
| HEADING-LEVEL | If expandable, `labelHeadingTag` is set |

## a!boxLayout

| Rule ID | What to Check |
|---|---|
| BOX-LABEL | If expandable (`isCollapsible: true`), `label` is set |
| HEADING-LEVEL | If expandable, `labelHeadingTag` is set |

## a!tabItem

| Rule ID | What to Check |
|---|---|
| TAB-LABEL | `label` is not null |

## a!pane

| Rule ID | What to Check |
|---|---|
| PANE-A11Y-TEXT | `accessibilityText` is set; does not contain reserved words (pane, main, navigation, section, form, search, header, footer, article, region) |

## a!cardLayout

| Rule ID | What to Check |
|---|---|
| CARD-LINK-NO-CONTROLS | If `link` is set, no child components use links, buttons, or inputs |
| CARD-LINK-NO-LABEL | If `link` is set, the link's `label` is null |
| CARD-SELECTED | If visual selection indicator is present, `accessibilityText: "Selected"` is set; removed when not selected |

## a!cardChoiceField

| Rule ID | What to Check |
|---|---|
| CARD-CHOICE-LABEL | `label` is set when more than one card in the group |

## a!cardGroupLayout

| Rule ID | What to Check |
|---|---|
| CARD-GROUP-LABEL | `label` is set when more than one card in the group |

## a!richTextDisplayField / a!richTextItem

| Rule ID | What to Check |
|---|---|
| LIST-SEMANTIC | Visual lists use `a!richTextBulletedList` or `a!richTextNumberedList` |
| LINK-INLINE-STYLE | Links in text use `linkStyle: "INLINE"` |
| HEADING-SEMANTIC | Text acting as headings does NOT use styled rich text — use `a!headingField` instead |

## Icons (a!richTextIcon, icons in links/buttons)

| Rule ID | What to Check |
|---|---|
| ICON-LINK-STANDALONE | Sole icon in a link: `altText` or `caption` is set (not both) |
| ICON-LINK-WITH-TEXT | Icon with text in a link: `altText`/`caption` only if text doesn't convey meaning |
| ICON-BUTTON-STANDALONE | Sole icon in a button: `accessibilityText` or `tooltip` on the button |
| ICON-BUTTON-WITH-TEXT | Icon with text in a button: `accessibilityText` only if label doesn't convey meaning |
| ICON-STANDALONE-INFO | Standalone informational icon: `altText`, `caption`, or `accessibilityText` is set |
| ICON-DECORATIVE | Decorative/redundant icon: `altText` and `caption` are both null |

## a!progressBarField

| Rule ID | What to Check |
|---|---|
| PROGRESS-LABEL | `label` is set |

## a!fileUploadField

| Rule ID | What to Check |
|---|---|
| FILE-UPLOAD-LABEL | `label` is set |
| FILE-UPLOAD-INSTRUCTIONS | `instructions` is set |

## Chart Components

Applies to: `a!barChartField`, `a!columnChartField`, `a!lineChartField`, `a!pieChartField`, `a!scatterChartField`, `a!areaChartField`

| Rule ID | What to Check |
|---|---|
| CHART-LABEL | `label` is set |

## a!formLayout

| Rule ID | What to Check |
|---|---|
| FORM-FOCUS | If important info precedes inputs, `focusOnFirstInput: false` or `skipAutoFocus: true` |
| FORM-REQUIRED-LEGEND | If any inputs have `required: true`, a visible legend ("Required fields are marked with an asterisk (*)") appears before the first input |

## a!stampField

| Rule ID | What to Check |
|---|---|
| STAMP-NO-TOOLTIP | `tooltip` and `helpTooltip` are null or absent |

## a!signatureField

| Rule ID | What to Check |
|---|---|
| SIGNATURE-ALTERNATIVE | An alternative keyboard-accessible method (checkbox, dropdown, etc.) must be provided alongside the signature component |

## a!dateTimeField

| Rule ID | What to Check |
|---|---|
| NO-DATETIME-FIELD | This component must not be used. Replace with separate `a!dateField` and `a!timeField`. |

## a!messageBanner

| Rule ID | What to Check |
|---|---|
| DYNAMIC-MESSAGE | `announceBehavior` parameter is set appropriately |

## Dynamic Content (showWhen-driven visibility)

| Rule ID | What to Check |
|---|---|
| DYNAMIC-CONTENT-ORDER | Content revealed via `showWhen` appears in tab order AFTER the triggering control, OR the trigger has `accessibilityText` warning of positioning, OR `a!messageBanner` notifies after reveal (flag for review) |

## Pagination Links (a!richTextIcon / icon links)

| Rule ID | What to Check |
|---|---|
| PAGINATION-INACTIVE-SILENT | When pagination links are inactive/non-applicable, `accessibilityText`, `altText`, and `caption` must all be null — inactive controls must be invisible to assistive tech |

## Simulated Grids (columnsLayout / sideBySideLayout mimicking tabular data)

| Rule ID | What to Check |
|---|---|
| SIMULATED-GRID-A11Y | Each "cell" in a layout-based grid must have `accessibilityText` indicating the column header and row header text (flag for review) |

## a!imageField, a!documentImage

| Rule ID | What to Check |
|---|---|
| IMAGE-OF-TEXT | Images must not contain embedded text unless they are logos or use fonts that cannot be rendered by a browser (flag for review) |

## Breadcrumbs (a!richTextDisplayField)

| Rule ID | What to Check |
|---|---|
| BREADCRUMB-A11Y | `accessibilityText` identifies the element as a breadcrumb and indicates the current (last) page |
