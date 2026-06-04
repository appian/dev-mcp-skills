# SAIL SideBySideLayout Usage Instructions

## Overview
SideBySideLayout arranges components horizontally. Use for component-level arrangements like icon+text, stamp+description, label+value. You can nest sideBySideLayouts and include multiple components within a single sideBySideItem, allowing you to stack and arrange components next to each other with perfect alignment.

---

## ⚠️ CRITICAL: Width Values for SideBySideItem

### ✅ VALID Width Values (ONLY These)
```
"AUTO"      - Fill remaining space (default)
"MINIMIZE"  - Shrink to fit content
"1X" to "10X" - Proportional widths
```

### ❌ INVALID Width Values (Runtime Errors)
**These are for a!columnLayout ONLY:**
```
"NARROW", "NARROW_PLUS", "MEDIUM", "MEDIUM_PLUS",
"WIDE", "WIDE_PLUS", "EXTRA_NARROW"  ❌ ALL WRONG
```

**Common Mistake:**
```sail
/* ❌ WRONG */
a!sideBySideItem(
  item: a!dropdownField(...),
  width: "NARROW"  /* ERROR */
)

/* ✅ CORRECT */
a!sideBySideItem(
  item: a!dropdownField(...),
  width: "MINIMIZE"
)
```

---

## What's Allowed in sideBySideItem

### ✅ ALLOWED:
- A single component (e.g., a!textField, a!richTextDisplayField)
- An array of components (multiple components stacked vertically within one item)
- A nested a!sideBySideLayout (the ONLY layout type allowed)

### ❌ NOT ALLOWED:
- Other layout types: cardLayout, columnsLayout, sectionLayout, boxLayout, etc.
- Use in grid columns

---

## Quick Reference

### Width Selection
- Icons/stamps: `"MINIMIZE"`
- Labels/short text: `"MINIMIZE"`
- Main content: `"AUTO"`
- Equal columns: `"1X"`, `"2X"`, etc.

### Vertical Alignment
- `"TOP"` (default), `"MIDDLE"`, `"BOTTOM"`

### Spacing
- `"STANDARD"` (default), `"NONE"`, `"DENSE"`, `"SPARSE"`
- ❌ Don't use `"less"` or `"more"`

---

## Common Patterns

### Icon + Text
```sail
a!sideBySideLayout(
  items: {
    a!sideBySideItem(
      item: a!richTextDisplayField(
        value: a!richTextIcon(icon: "check", color: "POSITIVE"),
        labelPosition: "COLLAPSED"
      ),
      width: "MINIMIZE"
    ),
    a!sideBySideItem(
      item: a!richTextDisplayField(
        value: "Task completed",
        labelPosition: "COLLAPSED"
      ),
      width: "AUTO"
    )
  },
  alignVertical: "MIDDLE"
)
```

### Stamp + Description
```sail
a!sideBySideLayout(
  items: {
    a!sideBySideItem(
      item: a!stampField(
        icon: "user",
        backgroundColor: "#3B82F6",
        size: "TINY",
        labelPosition: "COLLAPSED"
      ),
      width: "MINIMIZE"
    ),
    a!sideBySideItem(
      item: a!richTextDisplayField(
        labelPosition: "COLLAPSED",
        value: {
          a!richTextItem(text: "John Doe", style: "STRONG"),
          char(10),
          "Software Engineer"
        }
      ),
      width: "AUTO"
    )
  },
  alignVertical: "MIDDLE"
)
```

### Label + Value
```sail
a!sideBySideLayout(
  items: {
    a!sideBySideItem(
      item: a!richTextDisplayField(
        labelPosition: "COLLAPSED",
        value: a!richTextItem(text: "Status:", style: "STRONG")
      ),
      width: "MINIMIZE"
    ),
    a!sideBySideItem(
      item: a!tagField(
        labelPosition: "COLLAPSED",
        tags: a!tagItem(text: "Active", backgroundColor: "POSITIVE")
      ),
      width: "AUTO"
    )
  },
  alignVertical: "MIDDLE"
)
```

### Proportional Fields
```sail
a!sideBySideLayout(
  items: {
    a!sideBySideItem(
      item: a!textField(label: "First Name", value: local!firstName),
      width: "3X"
    ),
    a!sideBySideItem(
      item: a!textField(label: "M.I.", value: local!mi),
      width: "1X"
    ),
    a!sideBySideItem(
      item: a!textField(label: "Last Name", value: local!lastName),
      width: "3X"
    )
  }
)
```


### Multiple Components in One Item (Stacking)
Stack multiple components vertically within a single sideBySideItem by passing an array:
```sail
a!sideBySideLayout(
  items: {
    a!sideBySideItem(
      item: a!stampField(
        icon: "user",
        backgroundColor: "#3B82F6",
        size: "TINY",
        labelPosition: "COLLAPSED"
      ),
      width: "MINIMIZE"
    ),
    a!sideBySideItem(
      item: {
        a!richTextDisplayField(
          labelPosition: "COLLAPSED",
          value: a!richTextItem(text: "John Doe", style: "STRONG"),
          marginBelow: "NONE"
        ),
        a!richTextDisplayField(
          labelPosition: "COLLAPSED",
          value: a!richTextItem(text: "Software Engineer", color: "SECONDARY", size: "SMALL")
        )
      },
      width: "AUTO"
    )
  },
  alignVertical: "MIDDLE"
)
```

### Nested SideBySideLayout
Nest a sideBySideLayout inside a sideBySideItem for complex horizontal arrangements:
```sail
a!sideBySideLayout(
  items: {
    a!sideBySideItem(
      item: a!stampField(
        icon: "briefcase",
        backgroundColor: "#2C3E50",
        size: "TINY",
        labelPosition: "COLLAPSED"
      ),
      width: "MINIMIZE"
    ),
    a!sideBySideItem(
      item: a!sideBySideLayout(
        items: {
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextItem(text: "Project Alpha", style: "STRONG")
            ),
            width: "AUTO"
          ),
          a!sideBySideItem(
            item: a!tagField(
              labelPosition: "COLLAPSED",
              tags: a!tagItem(text: "Active", backgroundColor: "POSITIVE")
            ),
            width: "MINIMIZE"
          )
        },
        alignVertical: "MIDDLE",
        spacing: "DENSE"
      ),
      width: "AUTO"
    )
  },
  alignVertical: "MIDDLE"
)
```

### Combining Nesting and Stacking
Use both features together — an array of components where one is a nested sideBySideLayout:
```sail
a!sideBySideLayout(
  items: {
    a!sideBySideItem(
      item: a!stampField(
        icon: "clipboard",
        backgroundColor: "#8E44AD",
        size: "TINY",
        labelPosition: "COLLAPSED"
      ),
      width: "MINIMIZE"
    ),
    a!sideBySideItem(
      item: {
        a!sideBySideLayout(
          items: {
            a!sideBySideItem(
              item: a!richTextDisplayField(
                labelPosition: "COLLAPSED",
                value: a!richTextItem(text: "Task #1042", style: "STRONG"),
              ),
              width: "AUTO"
            ),
            a!sideBySideItem(
              item: a!tagField(
                labelPosition: "COLLAPSED",
                tags: a!tagItem(text: "In Progress", backgroundColor: "#F59E0B"),
              ),
              width: "MINIMIZE"
            )
          },
          alignVertical: "MIDDLE",
          spacing: "DENSE",
          marginBelow: "NONE"
        ),
        a!richTextDisplayField(
          labelPosition: "COLLAPSED",
          value: a!richTextItem(
            text: "Due: April 15, 2026",
            color: "SECONDARY",
            size: "SMALL"
          )
        )
      },
      width: "AUTO"
    )
  },
  alignVertical: "MIDDLE"
)
```

---

## When to Use What

### Use SideBySideLayout:
- Icon next to text
- Stamp with description
- Label and value pairs
- Small related UI elements
- Nested horizontal arrangements within a row (via nesting)
- Stacking multiple components alongside other items (via arrays)

### Use ColumnsLayout:
- Main page structure
- Content sections with margins
- Fixed-width columns

---

## VALIDATION CHECKLIST
- [ ] ⚠️ Width is `"AUTO"`, `"MINIMIZE"`, or `"1X"`-`"10X"` ONLY
- [ ] ❌ NOT using `"NARROW"`, `"MEDIUM"`, `"WIDE"` (columnLayout widths)
- [ ] ⚠️ Spacing is `"STANDARD"`, `"NONE"`, `"DENSE"`, or `"SPARSE"`
- [ ] ❌ NOT using `"less"` or `"more"` for spacing
- [ ] Only sideBySideLayout is allowed as a nested layout — no cardLayout, columnsLayout, etc.
