# Appian SAIL Accessibility Reference

This reference consolidates accessibility guidance for Appian SAIL interfaces. Use it as a lookup when auditing or building accessible interfaces.

## Source

All content here is derived from the Appian accessibility checklist and WCAG 2.2 guidelines.

---

## Accessibility Checklist Summary

The checklist defines success criteria across these categories. Each criterion specifies a SAIL Testing method (inspectable via component parameters), plus screen reader, keyboard, DOM, and visual testing approaches. The rules below focus on what can be verified programmatically from SAIL source or rendered component trees.

---

## Form Inputs

| Criterion | Rule |
|---|---|
| Every input must have a label | Set the `label` parameter on all inputs (enabled and disabled). The label must describe the input's purpose. |
| Placeholder text is not a substitute | Placeholder text alone must not convey important information (formats, ranges, lengths). |
| Labels must be persistently visible | Use `labelPosition` values other than `COLLAPSED` unless accessibility text provides the equivalent. |
| Accessible name matches visible label | The `label` parameter value must contain at least the same text as the visible label. |
| Duplicated controls need context | When multiple controls share the same name, provide distinguishing context via `accessibilityText`, icon `altText`, or by outputting repeated sections inside a labeled `a!sectionLayout` or `a!boxLayout`. |
| Checkbox/radio choice labels | `choiceLabels` must not contain null values on `a!checkboxField` or `a!radioButtonField`. |
| Checkbox/radio group label | When more than one option exists, the component-level `label` must be set. |
| Personal info autocomplete | Text fields accepting personal information must set `inputPurpose` appropriately. Do not set if accessible on public devices. |
| Toggle label | `a!toggleField` must have a non-null `choiceLabel`. |

---

## Validations

| Criterion | Rule |
|---|---|
| Required inputs | Set `required: true` on required inputs. |
| Use OOTB validation | All error messaging must use `validations`, `validationGroup`, and `validationMessage` parameters — not custom rich text. |
| Include input name in error | Error text should follow the pattern "[input name] requires a value." or similar. |

---

## Instructions

| Criterion | Rule |
|---|---|
| Use the instructions parameter | Visible instructions, expected formats, valid ranges, and input length guidance must use the `instructions` parameter on the input — not separate rich text. |

---

## Forms

| Criterion | Rule |
|---|---|
| Required field legend | Forms with asterisk-marked required inputs must have a visible legend before the first input: "Required fields are marked with an asterisk (*)". |
| Focus management | Set `focusOnFirstInput: false` on the form when important information precedes the first input. |
| Auto-populate re-entered info | Information a user must re-enter across form steps must be auto-populated or available for selection. |

---

## Grids

| Criterion | Rule |
|---|---|
| Grid label | `a!gridField` must have a `label` that summarizes the grid content. Can use `labelPosition: "COLLAPSED"` if preceded by a visible heading. |
| Column headers | Every `a!gridColumn` containing data must have a non-null `label`. Exception: columns with multiple controls whose purpose is self-evident. |
| Row header | Set `rowHeader` to the column position whose content uniquely identifies each row. |
| No empty spacing columns | Every `a!gridColumn` must have a valid `label` and `value`. |
| Grid instructions | Use the `instructions` parameter. Use "table" instead of "grid" in instruction text. |
| Row-selection controls warning | When controls above the grid enable on row selection, set `accessibilityText` on the grid to warn users. Use "table" instead of "grid" in the text. |
| Row ordering links target size | Links for moving rows must meet 24×24px target size or have sufficient spacing (3 spaces between links or use buttons). |
| Drag-and-drop alternative | Provide single-click row reordering (up/down links) as alternative to drag-and-drop. |

---

## Headings

| Criterion | Rule |
|---|---|
| Use semantic headings | Text headings must use `a!headingField` or a layout's `labelHeadingTag` — never styled rich text. |
| Appropriate heading levels | Set `headingTag` (on `a!headingField`) or `labelHeadingTag` (on section/box layouts) to reflect content hierarchy (H1–H6). |

---

## Lists

| Criterion | Rule |
|---|---|
| Semantic list markup | Visual lists must use `a!richTextBulletedList` or `a!richTextNumberedList` — not manual bullet characters or icon lists. |

---

## Section and Box Layouts

| Criterion | Rule |
|---|---|
| Expandable section label | Expandable `a!sectionLayout` must have both `label` and `labelHeadingTag` set. |
| Expandable box label | Expandable `a!boxLayout` must have both `label` and `labelHeadingTag` set. |

---

## Tab Layout

| Criterion | Rule |
|---|---|
| Tab labels | Each `a!tabItem` must have a non-null `label`. |

---

## Pane Layout

| Criterion | Rule |
|---|---|
| Pane accessibility text | Each `a!pane` must have `accessibilityText` set. The text must NOT contain these reserved words: pane, main, navigation, section, form, search, header, footer, article, region. |

---

## Cards

| Criterion | Rule |
|---|---|
| Linked card — no nested controls | If `link` is set on `a!cardLayout`, no child components may use links, buttons, or inputs. |
| Linked card — no link label | If `link` is set on a card, the link's `label` must be null. |
| Selection indicator | When color/decorative bar indicates the selected card, set `accessibilityText: "Selected"` on that card. Remove it when deselected. |
| Card choice group label | `a!cardChoiceField` must have a `label` when more than one card exists. |
| Card group layout label | `a!cardGroupLayout` must have a `label` when more than one card exists. |

---

## Links

| Criterion | Rule |
|---|---|
| Inline link differentiation | Links embedded in text must use `linkStyle: "INLINE"` or other visual differentiation (underline, font change, 3:1 contrast with surrounding text). |
| Rich text link selection state | When styling indicates a link is selected, use `accessibilityText: "Selected"` only if the rich text does NOT contain other content that conveys the same info. |

---

## Icons

| Criterion | Rule |
|---|---|
| Standalone icon in a link | Set `altText` or `caption` (not both) to describe the icon's meaning. |
| Icon with text in a link | Set `altText`/`caption` only if the link text doesn't already convey the icon's meaning. |
| Standalone icon in a button | Set `accessibilityText` or `tooltip` on the button. |
| Icon with text in a button | Set `accessibilityText` only if the button label doesn't already convey the icon's meaning. |
| Standalone informational icon | Set `altText`, `caption`, or `accessibilityText` to describe the icon. |
| Decorative/redundant icon | Both `altText` and `caption` must be null. |

---

## Breadcrumbs

| Criterion | Rule |
|---|---|
| Breadcrumb identification | Set `accessibilityText` on `a!richTextDisplayField` to identify the element as a breadcrumb and indicate the current (last) page. Example: "Breadcrumbs, Corporate is the current page". |

---

## Progress Bar

| Criterion | Rule |
|---|---|
| Label required | `a!progressBarField` must have a `label` set. |

---

## File Upload

| Criterion | Rule |
|---|---|
| Label required | `a!fileUploadField` must have a `label` set. |
| Instructions required | `a!fileUploadField` must have `instructions` set. |

---

## Charts

| Criterion | Rule |
|---|---|
| Label required | All chart components must have a `label` set. Can use `labelPosition: "COLLAPSED"` if preceded by a descriptive heading. |

---

## Modal Dialog

| Criterion | Rule |
|---|---|
| Focus management in dialogs | If important info precedes the first input in a dialog, set `focusOnFirstInput: false` on the form. |

---

## Stamps

| Criterion | Rule |
|---|---|
| No tooltip for important info | `a!stampField` must not use `tooltip` or `helpTooltip` to convey important information — these parameters must be null or absent. |

---

## Prohibited Components

| Criterion | Rule |
|---|---|
| No Date & Time component | `a!dateTimeField` must not be used. Replace with separate `a!dateField` and `a!timeField`. |

---

## Dynamic Content

| Criterion | Rule |
|---|---|
| Dynamically added content | Content added dynamically must appear in the tab order after the control that triggered it, OR users must be warned beforehand via `accessibilityText`, OR notified afterward via `a!messageBanner`. |
| Dynamic status messages | Use `a!messageBanner` with `announceBehavior` parameter for live status announcements. |

---

## Simulated Grids

| Criterion | Rule |
|---|---|
| Fake grid cells need context | Data displayed as a grid without `a!gridField` must have `accessibilityText` on each cell indicating the column header and row header text. |

---

## Color and Contrast (Manual Testing Required)

These cannot be checked programmatically from SAIL source but are part of the full checklist:

| Criterion | Requirement |
|---|---|
| Regular text contrast | 4.5:1 minimum between text and background. |
| Large text contrast | 3:1 minimum (18pt or 14pt bold). |
| Icon/image contrast | 3:1 minimum for informational graphics. |
| Selection state contrast | 3:1 between selected and unselected element colors. |
| Color not sole identifier | Color alone must not be the only means of identification. |

---

## Magnification (Manual Testing Required)

| Criterion | Requirement |
|---|---|
| 200% zoom | All text readable, functional, no overlap. |
| 400% zoom (1280px width) | No loss of content/functionality, no two-axis scrolling. Grids and complex images exempt. |

---

## Keyboard and Focus (Manual Testing Required)

| Criterion | Requirement |
|---|---|
| Link focus visibility | Rich text links must show a visual focus indicator (underline color change, focus ring). |
| Custom pagination spacing | Active adjacent pagination links must meet 24×24px target size or not overlap when 24px circles are centered on each. |
| Inactive pagination | Inactive/non-applicable links must not have accessibility text or icon alt text — they should be ignored by assistive tech. |

---

## Component-to-Rule Quick Lookup

For a compact mapping of which rules apply to which SAIL components, see: `component-checks.md`

---

## Design Patterns

The following design patterns have built-in accessibility considerations:

### Cards Pattern
- Use `a!cardGroupLayout` with `label` when displaying multiple related cards
- Avoid cards within cards (creates confusing border nesting)
- Use `padding: "STANDARD"` for breathing room
- For empty states within cards, provide clear messaging and a forward-flow action

### Grids Pattern
- Always set `label` and `rowHeader` on `a!gridField`
- Column headers required on all data columns
- Use `instructions` parameter for guidance text (say "table", not "grid")
- Provide keyboard alternatives for drag-and-drop row ordering

### Forms Pattern
- Group related inputs in labeled `a!sectionLayout` with appropriate `labelHeadingTag`
- Use `a!columnsLayout` for multi-column forms with `stackWhen` for responsive behavior
- Set `required: true` and use OOTB validation
- Include legend text for asterisk-marked fields

### Headings Pattern
- Use `a!headingField` with appropriate `headingTag` for standalone headings
- Use `labelHeadingTag` on `a!sectionLayout` and `a!boxLayout` for section headings
- Maintain proper hierarchy (H1 → H2 → H3, no skipped levels)

---

## How to Use This Reference

1. **During development** — When building a new interface, look up each component you're using and apply the relevant rules proactively.
2. **During audit** — Walk the rendered component tree and check each component against its applicable rules.
3. **When fixing defects** — Find the relevant rule by component type or Rule ID and apply the specified fix.
