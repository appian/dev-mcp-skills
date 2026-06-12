---
name: "appian-accessibility"
description: "Audit and fix accessibility issues in Appian SAIL interfaces using the DEV MCP. Covers WCAG compliance checks that can be performed programmatically by inspecting SAIL component trees — labels, accessibility text, heading structure, grid configuration, icon alt text, form validation patterns, and more. Use when reviewing interfaces for accessibility, fixing accessibility defects, or building new interfaces that must be accessible."
---

## Purpose

This skill enables programmatic accessibility auditing of Appian interfaces by inspecting the SAIL component tree returned by `testInterface`. It extracts the SAIL Testing checks from the Aurora Design System accessibility checklist — these are the checks that can be performed by examining SAIL parameters rather than requiring manual browser testing or screen reader verification.

## How It Works

1. **Retrieve the interface** — use `getInterface` to get the SAIL expression source
2. **Evaluate the interface** — use `testInterface` to render the component tree with test inputs
3. **Inspect the component tree** — walk the rendered output looking for accessibility violations against the rules in `#[[file:references/component-checks.md]]`
4. **Report findings** — list each violation with the component path, the rule violated, and the fix

When auditing, evaluate the interface with representative test inputs so that conditional components are rendered. If the interface has multiple states (create vs. edit, expanded vs. collapsed), evaluate each state separately.

## Rule Categories

The complete rules with Rule IDs, applicable components, and specific checks are in `#[[file:references/component-checks.md]]`. Here is what each category covers:

| Category | What to Check |
|---|---|
| **Form Inputs** | `label` set on all inputs, `labelPosition` handling, duplicated control context, `choiceLabels`, `inputPurpose`, toggle labels |
| **Validations** | `required: true` on mandatory fields, OOTB validation params used, input name included in error text |
| **Instructions** | Format/range guidance via `instructions` parameter, not separate rich text |
| **Grids** | Grid `label`, column headers, `rowHeader`, no empty spacer columns, instructions say "table" not "grid", selection `accessibilityText` |
| **Headings** | Semantic headings via `a!headingField` or `labelHeadingTag`, appropriate levels (H1–H6) |
| **Lists** | Visual lists use `a!richTextBulletedList`/`a!richTextNumberedList` |
| **Section/Box Layouts** | Expandable sections/boxes have `label` + `labelHeadingTag` |
| **Tabs** | Each `a!tabItem` has a `label` |
| **Panes** | `accessibilityText` set, no reserved words (pane, main, navigation, etc.) |
| **Cards** | Linked cards have no nested controls, no link label, selection state via `accessibilityText` |
| **Card Choice/Group** | `label` set when multiple cards in group |
| **Links** | `linkStyle: "INLINE"` for links embedded in text |
| **Icons** | `altText`/`caption` rules for standalone vs. with-text, in links vs. buttons, decorative icons null |
| **Progress Bar** | `label` set |
| **File Upload** | `label` and `instructions` set |
| **Charts** | `label` set |
| **Forms** | `focusOnFirstInput: false` when info precedes inputs, required-field legend present |
| **Modal Dialog** | `focusOnFirstInput: false` in dialogs with preceding content |
| **Stamps** | `tooltip`/`helpTooltip` must be null |
| **Prohibited** | `a!dateTimeField` must not be used |
| **Dynamic Content** | `a!messageBanner` with `announceBehavior`, content order after trigger |
| **Pagination** | Inactive links have null `accessibilityText`/`altText`/`caption` |
| **Simulated Grids** | Layout-based grids need `accessibilityText` per cell (flag for review) |
| **Signature** | Keyboard alternative alongside `a!signatureField` (flag for review) |
| **Images** | No embedded text in images (flag for review) |
| **Breadcrumbs** | `accessibilityText` identifies breadcrumb and current page |

## Audit Workflow

### Full Interface Audit

1. Get the interface UUID (from `listInterfaces` or provided by the user)
2. Get the interface definition with `getInterface` to read the SAIL source
3. Call `testInterface` with representative inputs to get the rendered component tree
4. Walk the component tree and check each component against the rules in `#[[file:references/component-checks.md]]`
5. For interfaces with multiple states, call `testInterface` multiple times with different inputs
6. Compile findings into a table: Rule ID | Component Path | Issue | Recommended Fix

### Quick Checks (Single Component Type)

When the user asks about a specific component or rule:
1. Search the rendered tree for that component type
2. Check only the relevant rules from the component-checks reference
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
- Include the input name in all validation/error messages
- Add a required-field legend when any inputs use `required: true`
- Ensure dynamically revealed content appears after its trigger in DOM order
- Set `accessibilityText` on breadcrumb rich text to identify it and the current page
- Provide a keyboard alternative alongside `a!signatureField`
- Set inactive pagination link parameters (`accessibilityText`, `altText`, `caption`) to null

## Limitations

These SAIL Testing checks cover what can be verified by inspecting component parameters. They do NOT cover:

- **Color contrast** — requires visual rendering and color analysis tooling
- **Target size** (24x24px minimum) — requires DOM measurement
- **Keyboard navigation order** — requires runtime keyboard testing
- **Screen reader announcements** — requires assistive technology testing
- **Visual inspection items** — placeholder text usage, focus visibility, content reflow at 200%/400% zoom

For complete WCAG 2.2 compliance, these programmatic checks must be supplemented with manual testing. See `#[[file:references/aurora-accessibility-reference.md]]` for the full checklist including manual testing categories.

## Reference: Aurora Design System

The canonical accessibility checklist is maintained in the Aurora Design System repository:

- **Repository**: https://github.com/appian-design/aurora
- **Key file**: `docs/accessibility/checklist.md` — full checklist with all testing methods
- **MCP server**: https://github.com/appian-design/aurora-mcp

To refresh your knowledge of the latest accessibility requirements:
```
https://raw.githubusercontent.com/appian-design/aurora/main/docs/accessibility/checklist.md
```

Use `web_fetch` on this URL when the rules in this skill may be outdated or when auditing against newly added criteria.

## Cross-References to Other Skills

### Always Relevant During Audits

- **appian-sail** — SAIL component parameters, layout hierarchy, corrected expressions. Covers grid patterns, link types, button styles that have accessibility rules.
- **appian-interfaces** — Interface restructuring, form organization, UX patterns (dashboard, form, summary view) that inform accessible layout choices.

### Relevant When Building Accessible Interfaces

- **appian-change-review** — Verify fixes landed correctly via `testInterface` (check `diagnostics.error`, inspect rendered tree).
- **appian-expressions** — Write/correct SAIL expressions for conditional `accessibilityText`, `showWhen` logic, dynamic labels.
- **appian-change-planning** — Scope systemic accessibility issues requiring multi-object changes.

### Occasionally Relevant

- **appian-supporting-objects** — Constants for centralized accessibility text values.
- **appian-security** — Role-based `showWhen` patterns affect what's visible in `testInterface` output.
- **appian-process-models** — Forms used as start forms or User Input Task forms; how to pass `ri!` inputs during `testInterface` evaluation.

### Not Typically Relevant

- **appian-record-types**, **appian-data-modeling**, **appian-web-apis**, **appian-sites**, **appian-expression-rules** — Backend/infrastructure skills that don't intersect with interface-level accessibility auditing.

## When You Need More

- For SAIL component syntax and parameters → activate **appian-sail**
- For interface design patterns → activate **appian-interfaces**
- For verifying changes → activate **appian-change-review**
- For writing SAIL expressions → activate **appian-expressions**
- For latest Appian docs → call `mcp__appian-docs__search_appian_knowledge_sources` with "accessibility" or component names
- For the full Aurora checklist → fetch from the GitHub URL above
