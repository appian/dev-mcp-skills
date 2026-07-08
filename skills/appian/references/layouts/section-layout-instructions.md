# a!sectionLayout Instructions

## Overview

`a!sectionLayout` groups content under a heading with optional collapse/expand functionality. It provides visual organization within forms and interfaces.

**Syntax:**
```sail
a!sectionLayout(
  label, 
  contents, 
  validations, 
  validationGroup, 
  isCollapsible, 
  isInitiallyCollapsed, 
  showWhen, 
  divider, 
  marginBelow, 
  accessibilityText, 
  labelIcon, 
  iconAltText, 
  labelSize, 
  labelHeadingTag, 
  labelColor, 
  dividerColor, 
  dividerWeight, 
  marginAbove
)
```

---

## ❌ CRITICAL: Common Parameter Mistakes

### Mistake 1: Using `showBorder` Parameter

**WRONG:**
```sail
a!sectionLayout(
  label: "Case Details",
  showBorder: true,  /* ❌ This parameter does NOT exist! */
  contents: {...}
)
```

**WHY IT FAILS:** `a!sectionLayout` does **NOT** have a `showBorder` parameter. This will cause an interface error.

**CORRECT ALTERNATIVES:**

**Option A: Use divider parameters for visual separation**
```sail
a!sectionLayout(
  label: "Case Details",
  divider: "ABOVE",           /* ✅ Adds line above section */
  dividerColor: "SECONDARY",  /* ✅ Sets divider color */
  dividerWeight: "THIN",      /* ✅ Sets divider thickness */
  contents: {...}
)
```

**Option B: Wrap in a!cardLayout if you need borders**
```sail
a!cardLayout(
  contents: {
    a!sectionLayout(
      label: "Case Details",
      contents: {...}
    )
  },
  showBorder: true,  /* ✅ cardLayout has showBorder */
  style: "NONE",
  padding: "STANDARD"
)
```

---

## Visual Separation Parameters

### Divider Parameters

Use these to add horizontal lines above or below the section:

```sail
a!sectionLayout(
  label: "Section Title",
  divider: "ABOVE",           /* "NONE" (default), "ABOVE", "BELOW" */
  dividerColor: "SECONDARY",  /* "STANDARD", "SECONDARY", "ACCENT" */
  dividerWeight: "THIN",      /* "THIN", "MEDIUM", "THICK" */
  contents: {...}
)
```

### Margin Parameters

Control spacing around the section:

```sail
a!sectionLayout(
  label: "Section Title",
  marginAbove: "STANDARD",   /* Spacing above */
  marginBelow: "STANDARD",   /* Spacing below */
  contents: {...}
)
```

**Valid margin values:** `"NONE"`, `"EVEN_LESS"`, `"LESS"`, `"STANDARD"` (default), `"MORE"`, `"EVEN_MORE"`

---

## Collapsible Sections

### Basic Collapsible Section

```sail
a!sectionLayout(
  label: "Advanced Settings",
  isCollapsible: true,              /* Makes section collapsible */
  isInitiallyCollapsed: true,       /* Starts collapsed */
  labelHeadingTag: "H3",            /* REQUIRED for accessibility when collapsible */
  contents: {
    a!textField(label: "Setting 1", ...),
    a!dropdownField(label: "Setting 2", ...)
  }
)
```

**IMPORTANT:** When `isCollapsible: true`, you **MUST** set `labelHeadingTag` for accessibility (screen readers).

### Collapsible with Icon

```sail
a!sectionLayout(
  label: "Case Details",
  labelIcon: "folder-open",         /* Icon next to label */
  iconAltText: "Case information",  /* Screen reader text for icon */
  isCollapsible: true,
  labelHeadingTag: "H2",
  contents: {...}
)
```

---

## Styling Options

### Label Styling

```sail
a!sectionLayout(
  label: "Important Information",
  labelSize: "LARGE",               /* "SMALL", "MEDIUM" (default), "LARGE", "EXTRA_LARGE" */
  labelColor: "ACCENT",             /* "STANDARD" (default), "ACCENT", "POSITIVE", "NEGATIVE", "SECONDARY" */
  labelIcon: "exclamation-circle",
  contents: {...}
)
```

---

## Common Patterns

### Pattern 1: Simple Section in Form

```sail
a!formLayout(
  titleBar: "Employee Details",
  contents: {
    a!sectionLayout(
      label: "Personal Information",
      contents: {
        a!textField(label: "First Name", value: local!firstName, saveInto: local!firstName),
        a!textField(label: "Last Name", value: local!lastName, saveInto: local!lastName)
      }
    ),
    a!sectionLayout(
      label: "Contact Information",
      marginAbove: "MORE",  /* Add spacing between sections */
      contents: {
        a!textField(label: "Email", value: local!email, saveInto: local!email),
        a!textField(label: "Phone", value: local!phone, saveInto: local!phone)
      }
    )
  }
)
```

### Pattern 2: Section with Dividers

```sail
a!sectionLayout(
  label: "Case History",
  divider: "ABOVE",
  dividerColor: "SECONDARY",
  dividerWeight: "THIN",
  marginAbove: "MORE",
  contents: {
    /* Timeline or history content */
  }
)
```

### Pattern 3: Collapsible Advanced Options

```sail
a!sectionLayout(
  label: "Advanced Options",
  labelIcon: "cog",
  iconAltText: "Configuration options",
  isCollapsible: true,
  isInitiallyCollapsed: true,  /* Hidden by default */
  labelHeadingTag: "H3",
  contents: {
    a!textField(label: "Custom Setting 1", ...),
    a!textField(label: "Custom Setting 2", ...)
  }
)
```

### Pattern 4: Section Inside Card (for borders)

```sail
a!cardLayout(
  contents: {
    a!sectionLayout(
      label: "Case Summary",
      labelColor: "ACCENT",
      labelSize: "LARGE",
      contents: {
        a!textField(label: "Title", value: local!title, readOnly: true),
        a!textField(label: "Status", value: local!status, readOnly: true)
      }
    )
  },
  showBorder: true,   /* Card provides the border */
  style: "NONE",
  padding: "STANDARD",
  shape: "ROUNDED"
)
```

---

## Decision Guide

| Scenario | Solution |
|----------|----------|
| Simple content grouping | Basic `a!sectionLayout` with `label` and `contents` |
| Visual separation between sections | Use `divider: "ABOVE"` and `marginAbove: "MORE"` |
| Need borders around section | Wrap in `a!cardLayout(showBorder: true)` |
| Optional/advanced content | Use `isCollapsible: true` with `labelHeadingTag` |
| Styled section header | Use `labelSize`, `labelColor`, `labelIcon` |

---

## Validation Messages

Section-level validation displays errors for all fields within the section:

```sail
a!sectionLayout(
  label: "Required Information",
  contents: {
    a!textField(label: "Field 1", ...),
    a!textField(label: "Field 2", ...)
  },
  validations: if(
    and(
      a!isNullOrEmpty(local!field1),
      a!isNullOrEmpty(local!field2)
    ),
    "At least one field in this section must be completed",
    null
  )
)
```

---

## Complete Example

```sail
a!formLayout(
  titleBar: "Project Configuration",
  contentsWidth: "MEDIUM",
  contents: {
    /* Basic Information Section */
    a!sectionLayout(
      label: "Basic Information",
      labelSize: "LARGE",
      contents: {
        a!textField(
          label: "Project Name",
          value: local!projectName,
          saveInto: local!projectName,
          required: true
        ),
        a!paragraphField(
          label: "Description",
          value: local!description,
          saveInto: local!description
        )
      }
    ),
    
    /* Advanced Settings Section - Collapsible */
    a!sectionLayout(
      label: "Advanced Settings",
      labelIcon: "cog",
      iconAltText: "Advanced configuration",
      isCollapsible: true,
      isInitiallyCollapsed: true,
      labelHeadingTag: "H2",
      divider: "ABOVE",
      dividerColor: "SECONDARY",
      marginAbove: "MORE",
      contents: {
        a!textField(
          label: "Custom Field 1",
          value: local!custom1,
          saveInto: local!custom1
        ),
        a!textField(
          label: "Custom Field 2",
          value: local!custom2,
          saveInto: local!custom2
        )
      }
    )
  },
  buttons: a!buttonLayout(
    primaryButtons: a!buttonWidget(
      label: "Save",
      submit: true
    )
  )
)
```

---

## Related Patterns

- **Form structure:** See [form-layout-instructions.md](form-layout-instructions.md)
- **Card borders:** See `a!cardLayout` in [component-reference.md](../component-reference.md)
- **Accessibility:** See [accessibility-reference.md](../accessibility-reference.md)
