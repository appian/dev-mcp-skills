# SAIL TabLayout Usage Instructions

## Overview
`a!tabLayout()` organizes content into selectable tabs with built-in navigation. Appian handles the tab switching logic automatically — no local variables or conditional visibility needed.

Each tab is defined with `a!tabItem()` which takes a label, optional icon, and contents.

---

## When to Use tabLayout vs Tab Patterns

### ✅ Use `a!tabLayout()` (the component) when:
- You need standard tab navigation with no custom styling
- You want built-in validation support (error icons on tabs with invalid inputs)
- You want automatic responsive behavior (arrow navigation when tabs overflow)
- You need tabs inside a form where sections can be completed in any order
- You want animations on transition states

### ✅ Use the manual tab pattern (`guidelines/patterns/tabs.md`) when:
- You need badges or counts on tab labels
- You need tabs integrated into a page header with a colored background
- You need custom tab styling (colors, sizes, decorative bars)

**Default to `a!tabLayout()` unless you need one of the custom pattern features above.**

---

## When to Use tabLayout vs Other Layouts

| Use | When |
|-----|------|
| `a!formLayout` alone | Simple form, no need to divide into sections |
| `a!wizardLayout` | Complex form with sequential steps |
| `a!tabLayout` inside `a!formLayout` | Complex form with independent sections completable in any order |
| `a!sectionLayout` inside `a!formLayout` | Complex form where all sections should be visible at once |

---

## Restrictions



---

## Quick Reference

### highlightColor
- `"ACCENT"` (default) — uses the application's accent color
- Any hex color — e.g., `"#2C3E50"`
- Hex with transparency — e.g., `"#2C3E5080"`

### contentsPadding
- `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"`

### Tab Labels
- Keep to 1-2 words
- Always pair icons with text labels

### ⚠️ Background
- tabLayout is transparent — it inherits the background of its parent

---

## Common Patterns

### Basic Tab Layout
```sail
a!tabLayout(
  tabs: {
    a!tabItem(
      label: "Overview",
      icon: "home",
      contents: {
        a!richTextDisplayField(
          labelPosition: "COLLAPSED",
          value: "Overview content goes here"
        )
      }
    ),
    a!tabItem(
      label: "Details",
      icon: "list",
      contents: {
        a!richTextDisplayField(
          labelPosition: "COLLAPSED",
          value: "Details content goes here"
        )
      }
    ),
    a!tabItem(
      label: "History",
      icon: "clock-o",
      contents: {
        a!richTextDisplayField(
          labelPosition: "COLLAPSED",
          value: "History content goes here"
        )
      }
    )
  },
  highlightColor: "ACCENT"
)
```

### Tabs Inside a Form
```sail
a!formLayout(
  titleBar: "Employee Record",
  contents: {
    a!tabLayout(
      tabs: {
        a!tabItem(
          label: "Personal",
          icon: "user",
          contents: {
            a!textField(label: "First Name", value: local!firstName, saveInto: local!firstName),
            a!textField(label: "Last Name", value: local!lastName, saveInto: local!lastName),
            a!dateField(label: "Date of Birth", value: local!dob, saveInto: local!dob)
          }
        ),
        a!tabItem(
          label: "Employment",
          icon: "briefcase",
          contents: {
            a!textField(label: "Title", value: local!title, saveInto: local!title),
            a!textField(label: "Department", value: local!department, saveInto: local!department)
          }
        ),
        a!tabItem(
          label: "Contact",
          icon: "envelope",
          contents: {
            a!textField(label: "Email", value: local!email, saveInto: local!email),
            a!textField(label: "Phone", value: local!phone, saveInto: local!phone)
          }
        )
      },
      highlightColor: "#2C3E50",
      contentsPadding: "STANDARD"
    )
  },
  buttons: a!buttonLayout(
    primaryButtons: a!buttonWidget(label: "Submit", style: "SOLID", submit: true)
  )
)
```

### Dashboard with Tabs
```sail
a!headerContentLayout(
  contents: {
    a!cardLayout(
      contents: {
        a!tabLayout(
          tabs: {
            a!tabItem(
              label: "Summary",
              contents: {
                /* KPI cards, charts, etc. */
                a!richTextDisplayField(
                  labelPosition: "COLLAPSED",
                  value: "Dashboard summary content"
                )
              }
            ),
            a!tabItem(
              label: "Projects",
              contents: {
                /* Project grid */
                a!richTextDisplayField(
                  labelPosition: "COLLAPSED",
                  value: "Projects grid content"
                )
              }
            ),
            a!tabItem(
              label: "Team",
              contents: {
                /* Team member list */
                a!richTextDisplayField(
                  labelPosition: "COLLAPSED",
                  value: "Team content"
                )
              }
            )
          }
        )
      },
      style: "NONE",
      shape: "ROUNDED",
      padding: "STANDARD"
    )
  },
  backgroundColor: "#F5F6F8"
)
```

---

## VALIDATION CHECKLIST
- [ ] Using `a!tabLayout()` with `a!tabItem()` (not manual tab pattern) unless custom styling is needed
- [ ] Not placed inside sideBySideLayout, editable grid, or read-only grid
- [ ] highlightColor is `"ACCENT"` or a valid hex color
- [ ] contentsPadding is `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"`, `"MORE"`, or `"EVEN_MORE"`
- [ ] marginAbove is `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"`, `"MORE"`, or `"EVEN_MORE"`
- [ ] marginBelow is `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"`, `"MORE"`, or `"EVEN_MORE"`
