---
name: "appian-accessibility"
description: "Audit and fix accessibility issues in Appian SAIL interfaces using the DEV MCP. Covers WCAG compliance checks that can be performed programmatically by inspecting SAIL component trees — labels, accessibility text, heading structure, grid configuration, icon alt text, form validation patterns, and more. Use when reviewing interfaces for accessibility, fixing accessibility defects, or building new interfaces that must be accessible."
---

## Purpose

This skill enables programmatic accessibility auditing of Appian interfaces by inspecting the SAIL component tree returned by `testInterface`. It extracts the SAIL Testing checks from the Aurora Design System accessibility checklist — these are the checks that can be performed by examining SAIL parameters rather than requiring manual browser testing or screen reader verification.

## How It Works

1. **Retrieve the interface** — use `getInterface` to get the SAIL expression source
2. **Evaluate the interface** — use `testInterface` to render the component tree with test inputs
3. **Inspect the component tree** — walk the rendered output looking for accessibility violations against the rules below
4. **Report findings** — list each violation with the component path, the rule violated, and the fix

When auditing, evaluate the interface with representative test inputs so that conditional components are rendered. If the interface has multiple states (create vs. edit, expanded vs. collapsed), evaluate each state separately.

## SAIL Testing Rules

These rules can be checked by inspecting SAIL parameters in the component tree. They are grouped by category.

### Form Inputs

| Rule ID | Component(s) | Check |
|---|---|---|
| INPUT-LABEL | All input components | The `label` parameter must not be null. Every enabled and disabled input must have a label that conveys its purpose. |
| INPUT-LABEL-MATCH | All input components with `labelPosition: "COLLAPSED"` | When `labelPosition` is `COLLAPSED`, the `label` parameter value must still be set and must contain at least the same string used for any visible label provided via rich text. |
| INPUT-DUPLICATES | All input components | When multiple controls share the same `label` text, check that `accessibilityText` is set on each to provide distinguishing context, OR that each repeated section uses a labeled `a!sectionLayout` or `a!boxLayout`. |
| CHECKBOX-CHOICE-LABELS | `a!checkboxField`, `a!radioButtonField` | The `choiceLabels` parameter must not contain null values. Each individual checkbox/radio button must have a label. |
| CHECKBOX-GROUP-LABEL | `a!checkboxField`, `a!radioButtonField` | When more than one checkbox or radio button is present in the group, the component-level `label` parameter must be set. |
| INPUT-PURPOSE | `a!textField` | Text fields accepting personal information (name, email, phone, address) should have the `inputPurpose` parameter set appropriately. |
| TOGGLE-LABEL | `a!toggleField` | The `choiceLabel` parameter must not be null. |

### Validations

| Rule ID | Component(s) | Check |
|---|---|---|
| REQUIRED-PARAM | All input components accepting user data | Inputs that are required must have `required: true` set. |
| VALIDATION-OOTB | All input components | Use built-in `validations`, `validationGroup`, and `validationMessage` parameters for error messaging rather than custom rich text error display. |

### Instructions

| Rule ID | Component(s) | Check |
|---|---|---|
| INPUT-INSTRUCTIONS | All input components | Visible instructions, expected formats, valid ranges, and input length guidance must be provided via the `instructions` parameter, not via separate rich text components. |

### Grids

| Rule ID | Component(s) | Check |
|---|---|---|
| GRID-LABEL | `a!gridField` | The `label` parameter must be set to a value that summarizes or describes the grid content. |
| GRID-COLUMN-HEADER | `a!gridColumn` | Each grid column that contains data must have a non-null `label` parameter. |
| GRID-ROW-HEADER | `a!gridField` | The `rowHeader` parameter should be set to the position of the column whose content uniquely identifies each row. |
| GRID-EMPTY-COLUMNS | `a!gridColumn` | Empty columns must not be used for spacing — every `a!gridColumn` must have a valid `label` and `value`. |
| GRID-INSTRUCTIONS | `a!gridField` | Grid instructions must use the `instructions` parameter. The text must not contain the word "grid" — use "table" instead. |
| GRID-SELECTION-A11Y | `a!gridField` | When controls above the grid become enabled based on row selection, `accessibilityText` must be set on the grid to warn users of this behavior prior to interaction. The text must not contain the word "grid" — use "table" instead. |

### Headings

| Rule ID | Component(s) | Check |
|---|---|---|
| HEADING-SEMANTIC | Text acting as headings | Text headings must use `a!headingField` or a layout with `labelHeadingTag` parameter — not styled rich text. |
| HEADING-LEVEL | `a!headingField`, layouts with `labelHeadingTag` | The `headingTag` or `labelHeadingTag` parameter must be set to an appropriate level reflecting content hierarchy. |

### Lists

| Rule ID | Component(s) | Check |
|---|---|---|
| LIST-SEMANTIC | `a!richTextDisplayField` | Visual lists (bullets, numbered items) must use `a!richTextBulletedList` or `a!richTextNumberedList`, not manual bullet characters or icons. |

### Section and Box Layouts

| Rule ID | Component(s) | Check |
|---|---|---|
| SECTION-LABEL | `a!sectionLayout` (expandable) | Expandable sections must have a `label` parameter set AND a `labelHeadingTag` parameter value. |
| BOX-LABEL | `a!boxLayout` (expandable) | Expandable boxes must have a `label` parameter set AND a `labelHeadingTag` parameter value. |

### Tab Layout

| Rule ID | Component(s) | Check |
|---|---|---|
| TAB-LABEL | `a!tabItem` | Each tab must have a non-null `label` parameter. |

### Pane Layout

| Rule ID | Component(s) | Check |
|---|---|---|
| PANE-A11Y-TEXT | `a!pane` | Each pane must have `accessibilityText` set. The text must not contain: pane, main, navigation, section, form, search, header, footer, article, or region. |

### Cards

| Rule ID | Component(s) | Check |
|---|---|---|
| CARD-LINK-NO-CONTROLS | `a!cardLayout` | When a `link` parameter is set on a card, no other components within the card may use a link, button, or input. |
| CARD-LINK-NO-LABEL | `a!cardLayout` | When a `link` parameter is set on a card, the link's `label` parameter must be null. |
| CARD-SELECTED | `a!cardLayout` | When color or a decorative bar indicates selection, `accessibilityText` must be set to "Selected" on the selected card. Unselected cards must NOT have "Selected" accessibility text. |

### Card Choice and Card Group

| Rule ID | Component(s) | Check |
|---|---|---|
| CARD-CHOICE-LABEL | `a!cardChoiceField` | Must have a `label` parameter when more than one card is present in the group. |
| CARD-GROUP-LABEL | `a!cardGroupLayout` | Must have a `label` parameter when more than one card is present in the group. |

### Links

| Rule ID | Component(s) | Check |
|---|---|---|
| LINK-INLINE-STYLE | `a!richTextItem` (with link) | Links embedded in text must use `linkStyle: "INLINE"` or other formatting to differentiate from surrounding text. |

### Icons

| Rule ID | Component(s) | Check |
|---|---|---|
| ICON-LINK-STANDALONE | Icon as sole element in a link | The `altText` or `caption` parameter must provide a text alternative that conveys the icon's meaning. Do not set both `altText` and `caption` on the same icon. |
| ICON-LINK-WITH-TEXT | Icon with text in a link | The `altText` or `caption` parameter should be set only if the link text does not already convey the icon's meaning. |
| ICON-BUTTON-STANDALONE | Icon as sole element in a button | The button must have `accessibilityText` or `tooltip` set to provide a text alternative. |
| ICON-BUTTON-WITH-TEXT | Icon with text in a button | The button should have `accessibilityText` set only if the button label does not already convey the icon's meaning. |
| ICON-STANDALONE-INFO | Standalone icon conveying information | The `altText`, `caption`, or `accessibilityText` parameter must provide a text alternative. |
| ICON-DECORATIVE | Decorative or redundant icon | The `altText` and `caption` parameters must be null — do not provide text alternatives for purely decorative icons. |

### Progress Bar

| Rule ID | Component(s) | Check |
|---|---|---|
| PROGRESS-LABEL | `a!progressBarField` | The `label` parameter must be set. |

### File Upload

| Rule ID | Component(s) | Check |
|---|---|---|
| FILE-UPLOAD-LABEL | `a!fileUploadField` | The `label` parameter must be set. |
| FILE-UPLOAD-INSTRUCTIONS | `a!fileUploadField` | The `instructions` parameter must be set with visible instruction text. |

### Charts

| Rule ID | Component(s) | Check |
|---|---|---|
| CHART-LABEL | Chart components | The `label` parameter must be set. |

### Forms

| Rule ID | Component(s) | Check |
|---|---|---|
| FORM-FOCUS | `a!formLayout` | If important information exists before the first input, set `focusOnFirstInput: false` (or `skipAutoFocus: true`) to prevent focus from skipping past it. |

### Modal Dialog

| Rule ID | Component(s) | Check |
|---|---|---|
| DIALOG-FOCUS | `a!formLayout` in dialog context | If important information exists before the first input in a dialog, set `focusOnFirstInput: false`. |

### Stamps

| Rule ID | Component(s) | Check |
|---|---|---|
| STAMP-NO-TOOLTIP | `a!stampField` | The `tooltip` and `helpTooltip` parameters must not convey important information — they must be null or absent. |

### Prohibited Components

| Rule ID | Component(s) | Check |
|---|---|---|
| NO-DATETIME-FIELD | `a!dateTimeField` | The Date & Time component must not be used. Use separate `a!dateField` and `a!timeField` instead. |

### Dynamic Content

| Rule ID | Component(s) | Check |
|---|---|---|
| DYNAMIC-MESSAGE | `a!messageBanner` | Dynamic status messages must use `a!messageBanner` with an appropriate `announceBehavior` parameter value. |

## Audit Workflow

### Full Interface Audit

1. Get the interface UUID (from `listInterfaces` or provided by the user)
2. Get the interface definition with `getInterface` to read the SAIL source
3. Call `testInterface` with representative inputs to get the rendered component tree
4. Walk the component tree and check each component against the applicable rules above
5. For interfaces with multiple states, call `testInterface` multiple times with different inputs
6. Compile findings into a table: Rule ID | Component Path | Issue | Recommended Fix

### Quick Checks (Single Component Type)

When the user asks about a specific component or rule:
1. Search the rendered tree for that component type
2. Check only the relevant rules
3. Report pass/fail with specifics

### Writing Accessible Interfaces

When creating new interfaces, apply these rules proactively:
- Set `label` on every input, grid, progress bar, file upload, and chart
- Set `accessibilityText` on panes and grids with row-selection behavior
- Use `a!headingField` or `labelHeadingTag` for headings — never styled rich text
- Use `a!richTextBulletedList`/`a!richTextNumberedList` for lists
- Provide `instructions` on inputs that need format guidance
- Set `altText` or `caption` on icons used standalone in links/buttons
- Never use `a!dateTimeField`
- Set `required: true` on mandatory inputs
- Use OOTB validation parameters instead of custom error display

## Limitations

These SAIL Testing checks cover what can be verified by inspecting component parameters. They do NOT cover:

- **Color contrast** — requires visual rendering and color analysis tooling (use Colour Contrast Analyser or axe DevTools)
- **Target size** (24x24px minimum) — requires DOM measurement
- **Keyboard navigation order** — requires runtime keyboard testing
- **Screen reader announcements** — requires assistive technology testing
- **Visual inspection items** — placeholder text usage, focus visibility, content reflow at 200%/400% zoom

For complete WCAG 2.2 compliance, these programmatic checks must be supplemented with manual testing. The full checklist including all testing methods is maintained in the Aurora Design System repository.

## Reference: Aurora Design System

The canonical accessibility checklist and design patterns are maintained in the Aurora Design System repository:

**Repository**: https://github.com/appian-design/aurora

Key files:
- `docs/accessibility/checklist.md` — full accessibility checklist with all testing methods
- `docs/components/` — component-specific design guidance including accessibility notes
- `docs/layouts/` — layout patterns with accessibility considerations

To refresh your knowledge of the latest accessibility requirements, fetch the raw checklist from the repository:

```
https://raw.githubusercontent.com/appian-design/aurora/main/docs/accessibility/checklist.md
```

Use `web_fetch` on this URL to get the latest version when the rules in this skill may be outdated or when auditing against newly added criteria.

## Cross-References to Other Skills

Accessibility auditing intersects with several other Appian skills. Activate them when you need deeper guidance in their domain:

### Always Relevant During Audits

- **appian-sail** — Activate when you need to understand SAIL component parameters, layout hierarchy, or write corrected expressions. The SAIL skill's `references/components/` folder contains per-component parameter details that help you understand which parameters are available for accessibility fixes. It also covers grid patterns, link types, and button styles — all of which have accessibility rules.

- **appian-interfaces** — Activate when fixing accessibility issues requires restructuring an interface — changing form organization, splitting a form into sections, adding inputs, or redesigning a summary view. The interfaces skill covers interface object lifecycle (create/update/inputs/expression) and UX patterns (dashboard, form, summary view) that inform accessible layout choices.

### Relevant When Building Accessible Interfaces From Scratch

- **appian-change-review** — Activate after applying accessibility fixes to verify the changes landed correctly. Its `testInterface` verification pattern (check `diagnostics.error`, inspect rendered tree) is the same mechanism used by this skill's audit workflow. Use it to confirm fixes didn't introduce rendering failures.

- **appian-expressions** — Activate when accessibility fixes require writing or correcting SAIL expressions — for example, building conditional `accessibilityText` values, `showWhen` logic for accessible patterns, or dynamic label text. Covers expression syntax, operators, null handling, and the `recordType!` reference format.

- **appian-change-planning** — Activate when an accessibility audit reveals systemic issues requiring multi-object changes (e.g., all interfaces in an application need heading restructuring, or a missing expression rule should be created to centralize accessibility text logic). The planning skill helps scope and order those changes.

### Occasionally Relevant

- **appian-supporting-objects** — Activate when accessibility fixes require creating constants (e.g., centralized accessibility text values) or organizing documents (e.g., accessibility testing documentation uploaded to a folder).

- **appian-security** — Activate when accessibility checks involve role-based visibility patterns. The `showWhen` expressions that conditionally show/hide components for different roles can affect what the auditor sees in `testInterface` output — you may need to evaluate the interface with different user contexts.

- **appian-process-models** — Activate when accessibility issues relate to forms used as start forms or User Input Task forms. The process model skill covers how interfaces connect to process models, which affects how `ri!record`, `ri!isUpdate`, and `ri!cancel` inputs should be passed during `testInterface` evaluation.

### Not Typically Relevant

- **appian-record-types**, **appian-data-modeling**, **appian-web-apis**, **appian-sites**, **appian-expression-rules** — These skills don't directly intersect with accessibility auditing. Record types and data modeling deal with backend data structure. Web APIs deal with HTTP endpoints. Sites deal with page organization and navigation. Expression rules deal with reusable logic encapsulation. Accessibility is checked at the interface level via the rendered component tree.

## When You Need More

- For SAIL component syntax and parameters, activate the **appian-sail** skill
- For interface design patterns and UX structure, activate the **appian-interfaces** skill
- For verifying changes after applying fixes, activate the **appian-change-review** skill
- For writing or fixing SAIL expressions, activate the **appian-expressions** skill
- For the latest Appian accessibility documentation, call `mcp__appian-docs__search_appian_knowledge_sources` with queries like "accessibility", "WCAG", or specific component names
- For the full Aurora checklist including manual testing methods, fetch the latest from the GitHub repository URL above
