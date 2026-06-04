# SAIL Button Usage Instructions

## Overview
Buttons trigger actions in SAIL interfaces. Buttons must always be placed inside a button container — never standalone.

**Two valid containers — use the right one:**
- `a!buttonLayout(primaryButtons: ..., secondaryButtons: ...)` — **form submission buttons only**, placed in `a!formLayout(buttons: ...)`. This is the correct wrapper for Submit/Cancel buttons at the bottom of forms.
- `a!buttonArrayLayout(buttons: ...)` — **all other buttons** (inline actions, non-submission buttons within interface body). Appian explicitly states this is "for buttons within interfaces rather than for submission buttons at the bottom of forms."

---

## ⚠️ Valid Style Values (MEMORIZE THESE)

**ONLY these 4 styles exist in SAIL:**
- `"OUTLINE"` - Bordered button (default, secondary actions)
- `"GHOST"` - Minimal styling (tertiary actions)
- `"LINK"` - Text-only link style
- `"SOLID"` - Filled button (primary actions)

### ❌ INVALID STYLES (DO NOT USE):
- `"PRIMARY"` ❌ (use `style: "SOLID"` + `color: "ACCENT"` instead)
- `"SECONDARY"` ❌ (use `style: "OUTLINE"` instead)
- `"DEFAULT"` ❌ (these are from React/Material-UI, not SAIL!)
- `"CONTAINED"` ❌
- `"TEXT"` ❌

---

## Button Parameters Quick Reference

| Parameter | Valid Values | Notes |
|-----------|--------------|-------|
| `label` | Text string | Button text |
| `style` | `"OUTLINE"`, `"GHOST"`, `"LINK"`, `"SOLID"` | **Only these 4!** |
| `color` | `"ACCENT"`, `"SECONDARY"`, `"NEGATIVE"`, hex codes (#RRGGBB) | For prominence/semantic meaning |
| `size` | `"SMALL"`, `"STANDARD"`, `"LARGE"` | |
| `icon` | Valid icon alias | MUST look up in: `references/rich-text-icon-aliases.md` |
| `saveInto` | Variable/expression | Action to perform |
| `submit` | Boolean | Submit form on click |
| `loadingIndicator` | Boolean | Shows spinner during processing |
| `disabled` | Boolean | Disables button |
| `showWhen` | Boolean | Conditional visibility |

---

## Common Button Patterns

### Primary Action Button
```sail
a!buttonArrayLayout(
  buttons: a!buttonWidget(
    label: "Save",
    style: "SOLID",
    color: "ACCENT",
    saveInto: {}
  )
)
```

### Secondary Action Button
```sail
a!buttonArrayLayout(
  buttons: a!buttonWidget(
    label: "Cancel",
    style: "OUTLINE",
    saveInto: {}
  )
)
```

### Tertiary/Link Button
```sail
a!buttonArrayLayout(
  buttons: a!buttonWidget(
    label: "Learn More",
    style: "LINK",
    saveInto: {}
  )
)
```

### Destructive Action Button
```sail
a!buttonArrayLayout(
  buttons: a!buttonWidget(
    label: "Delete",
    icon: "trash",
    style: "SOLID",
    color: "NEGATIVE",  /* Red for destructive actions */
    saveInto: {}
  )
)
```

### Button with Icon
```sail
a!buttonArrayLayout(
  buttons: a!buttonWidget(
    label: "Download Report",
    icon: "download",
    style: "OUTLINE",
    saveInto: {}
  )
)
```

### Multiple Buttons in Array
```sail
a!buttonArrayLayout(
  buttons: {
    a!buttonWidget(
      label: "Save",
      style: "SOLID",
      color: "ACCENT",
      saveInto: {}
    ),
    a!buttonWidget(
      label: "Cancel",
      style: "OUTLINE",
      saveInto: {}
    ),
    a!buttonWidget(
      label: "Reset",
      style: "LINK",
      saveInto: {}
    )
  },
  align: "START"
)
```

### Conditional Button (Show When)
```sail
a!buttonArrayLayout(
  buttons: {
    a!buttonWidget(
      label: "Save Draft",
      style: "OUTLINE",
      showWhen: local!isDraft,
      saveInto: {}
    ),
    a!buttonWidget(
      label: "Submit",
      style: "SOLID",
      color: "ACCENT",
      saveInto: {}
    )
  }
)
```

---

## ButtonArrayLayout Requirements

### Key Parameters
- `buttons`: Single button or array of buttons (required)
- `align`: `"START"`, `"CENTER"`, `"END"` (default: `"START"`)
- `marginBelow`: Standard margin options (`"NONE"`, `"STANDARD"`, `"MORE"`, etc.)

### Critical Rules
- ❌ Buttons CANNOT exist standalone — always use a container
- ✅ Use `a!buttonArrayLayout` for inline/body buttons; use `a!buttonLayout` for form submission buttons in `a!formLayout(buttons: ...)`
- ✅ Set `marginBelow: "NONE"` when inside other components (cards, sideBySide, etc.)
- ✅ Use `align` to position buttons within the layout

### Example: Buttons Inside Card
```sail
a!cardLayout(
  contents: {
    a!richTextDisplayField(...),
    a!buttonArrayLayout(
      buttons: a!buttonWidget(
        label: "Take Action",
        style: "SOLID",
        color: "ACCENT",
        saveInto: {}
      ),
      marginBelow: "NONE"  /* Important: no extra margin at bottom of card */
    )
  }
)
```

---

## Style-to-Purpose Mapping

**If you want this look** → **Use this SAIL style:**

| Desired Look | SAIL Implementation |
|--------------|---------------------|
| Primary button (filled, prominent) | `style: "SOLID"` + `color: "ACCENT"` |
| Secondary button (outlined) | `style: "OUTLINE"` |
| Tertiary button (minimal) | `style: "GHOST"` |
| Link-style button | `style: "LINK"` |
| Danger/destructive button | `style: "SOLID"` + `color: "NEGATIVE"` |
| Success button | `style: "SOLID"` + `color: "#059669"` |

---

## Validation Checklist

Before finalizing button code:

- [ ] Button `style` is one of: `"OUTLINE"`, `"GHOST"`, `"LINK"`, `"SOLID"`
- [ ] Button is in a container: `a!buttonArrayLayout` for inline buttons, `a!buttonLayout` for form submission buttons
- [ ] If using custom color, it's `"ACCENT"` or valid 6-character hex code
- [ ] If using icon, it's a valid icon alias found in `references/rich-text-icon-aliases.md`
- [ ] `marginBelow: "NONE"` set on buttonArrayLayout when inside other components
- [ ] Size is one of: `"SMALL"`, `"STANDARD"`, `"LARGE"` (if specified)

---

## Common Mistakes to Avoid

❌ **WRONG:**
```sail
/* Using invalid styles */
a!buttonWidget(style: "PRIMARY")      /* NO - doesn't exist */
a!buttonWidget(style: "SECONDARY")    /* NO - doesn't exist */

/* Button without wrapper */
a!buttonWidget(label: "Click Me")     /* NO - must be in a container */

/* Multiple standalone buttons */
{
  a!buttonWidget(label: "Save"),
  a!buttonWidget(label: "Cancel")     /* NO - wrap in a!buttonArrayLayout or a!buttonLayout */
}

/* Using buttonArrayLayout for form submission buttons */
a!formLayout(
  buttons: a!buttonArrayLayout(...)   /* NO - form buttons must use a!buttonLayout */
)
```

✅ **CORRECT:**
```sail
/* Using valid styles */
a!buttonArrayLayout(
  buttons: {
    a!buttonWidget(
      label: "Save",
      style: "SOLID",               /* YES - valid style */
      color: "ACCENT"               /* YES - for prominence */
    ),
    a!buttonWidget(
      label: "Cancel",
      style: "OUTLINE"              /* YES - valid style */
    )
  }
)
```

---

## Quick Reference: Framework Comparison

If you're familiar with other UI frameworks, note these differences:

| Other Frameworks | SAIL Equivalent |
|------------------|-----------------|
| `variant="contained"` (Material-UI) | `style: "SOLID"` |
| `variant="outlined"` (Material-UI) | `style: "OUTLINE"` |
| `variant="text"` (Material-UI) | `style: "LINK"` |
| `type="primary"` (Ant Design) | `style: "SOLID"` + `color: "ACCENT"` |
| `type="default"` (Ant Design) | `style: "OUTLINE"` |
| `type="link"` (Ant Design) | `style: "LINK"` |

**Remember:** SAIL has its own style system - don't assume values from other frameworks will work!
