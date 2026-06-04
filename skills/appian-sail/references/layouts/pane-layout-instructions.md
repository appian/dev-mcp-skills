# SAIL PaneLayout Usage Instructions

## Overview
PaneLayout creates full-height, independently-scrolling vertical panes that fill the entire browser window. It's designed for application-style interfaces where each pane serves a distinct purpose and needs independent scrolling behavior.

## When to Use PaneLayout

### ✅ Use PaneLayout For:
- **Application-style interfaces** - File managers, admin consoles, IDEs
- **Master-detail views** - List + detail, navigation + content
- **Multi-panel dashboards** - Sidebar navigation + main content + detail panel
- **Full-height applications** - Apps that should fill the browser window
- **Independent scrolling needs** - Each pane scrolls separately
- **Desktop-like experiences** - Traditional desktop application layouts

### PaneLayout Characteristics:
- Panes fill full browser height (100vh)
- Each pane scrolls independently
- No built-in form submission handling
- Flexible width distribution
- Individual pane background colors
- Optimal for complex, multi-section interfaces

```sail
/* GOOD - Application-style interface */
a!paneLayout(
  panes: {
    a!pane(
      width: "NARROW",
      backgroundColor: "#f8f9fa",
      contents: {
        a!sectionLayout(
          label: "Navigation",
          contents: {
            a!linkField(links: a!dynamicLink(label: "Dashboard")),
            a!linkField(links: a!dynamicLink(label: "Reports")),
            a!linkField(links: a!dynamicLink(label: "Settings"))
          }
        )
      }
    ),
    a!pane(
      width: "AUTO",  /* Takes remaining space */
      contents: {
        a!gridField(...)  /* Main content */
      }
    )
  }
)
```

## ✅ CRITICAL RULE: Don't nest pane layouts
- PaneLayouts must either be the top-level layout in a UI -or- be the top-level contents of a headerContentLayout
- PaneLayouts cannot be nested within any layout other than headerContentLayout
- Panes cannot contain headerContentLayouts or formLayouts

## Core PaneLayout Structure

### Required Parameters
- **panes**: List of a!pane() components (minimum 2, maximum 3 panes)

### Individual Pane Parameters
- **contents**: Components and layouts within the pane
- **width**: Pane width constraint
- **backgroundColor**: Individual pane background color
- **padding**: Internal spacing within pane

## Pane Width Strategy

### Width Values and Usage
- **AUTO**: Takes all remaining horizontal space (fluid)
- **EXTRA_NARROW**: ~80px (icons, minimal controls)
- **NARROW**: ~240px (navigation menus, sidebars)
- **NARROW_PLUS**: ~320px (detailed sidebars)
- **MEDIUM**: ~400px (moderate content panels)
- **MEDIUM_PLUS**: ~560px (detailed content)
- **WIDE**: ~800px (main content areas)
- **WIDE_PLUS**: ~1120px (wide content, rare in panes)

### ✅ CRITICAL RULE: Always Include One AUTO Width Pane
At least one pane must have `width: "AUTO"` to create fluid layout and fill remaining space.

```sail
/* CORRECT - One AUTO pane for fluid layout */
a!paneLayout(
  panes: {
    a!pane(width: "NARROW", ...),      /* Fixed sidebar */
    a!pane(width: "AUTO", ...),        /* Fluid main content */
    a!pane(width: "MEDIUM", ...)       /* Fixed detail panel */
  }
)
```

## Common Pane Layout Patterns

### 1. Two-Pane: Navigation + Content
```sail
a!paneLayout(
  panes: {
    a!pane(
      width: "NARROW",
      backgroundColor: "#f8f9fa",
      padding: "STANDARD",
      contents: {
        /* Navigation, filters, or tools */
        a!sectionLayout(label: "Filters", contents: {...}),
        a!sectionLayout(label: "Actions", contents: {...})
      }
    ),
    a!pane(
      width: "AUTO",
      contents: {
        /* Main content area */
        a!gridField(...),
        a!columnChartField(...)
      }
    )
  }
)
```

### 2. Three-Pane: Navigation + Content + Detail
```sail
a!paneLayout(
  panes: {
    a!pane(
      width: "NARROW",
      backgroundColor: "#2c3e50",
      contents: {
        /* Left navigation */
        a!richTextDisplayField(
          value: a!richTextItem(text: "App Menu", color: "#FFFFFF")
        )
      }
    ),
    a!pane(
      width: "AUTO",
      contents: {
        /* Main content list/grid */
        a!gridField(...)
      }
    ),
    a!pane(
      width: "MEDIUM",
      backgroundColor: "#ecf0f1",
      contents: {
        /* Detail panel */
        a!cardLayout(
          contents: {
            a!richTextDisplayField(value: "Item Details"),
            a!textField(...)
          }
        )
      }
    )
  }
)
```

### 3. Panes Below Header: headerContentLayout + paneLayout
```sail
a!headerContentLayout(
  header: { /* Page title header bar */
    a!cardLayout(
      contents: {
        a!headingField(
          text: "Prerequisite Verification Review",
          size: "LARGE",
          headingTag: "H1",
          fontWeight: "BOLD",
          color: "#FFFFFF",
          marginBelow: "NONE"
        )
      },
      height: "AUTO",
      style: "#1E40AF",
      padding: "MORE",
      marginBelow: "NONE",
      showBorder: false
    )
  },
  contents: a!paneLayout( /* Pane layout in headerContentLayout contents */
    panes: {
      a!pane(
        contents: {
          /* Side Pane Contents */
        },
        width: "NARROW_PLUS"
      ),
      a!pane(
        contents: {
          /* Main Pane Contents */
        },
        width: "AUTO"
      )
    }
  ),
  contentsPadding: "NONE"
)
```

## Common Anti-Patterns to Avoid

### ❌ No AUTO Width Pane
```sail
/* WRONG - All fixed widths, no fluid layout */
a!paneLayout(
  panes: {
    a!pane(width: "NARROW", ...),
    a!pane(width: "WIDE", ...)     /* Should be AUTO */
  }
)
```

### ❌ Using PaneLayout for Simple Forms
```sail
/* WRONG - Unnecessary complexity for a form */
a!paneLayout(
  panes: {
    a!pane(
      width: "AUTO",
      contents: {
        a!textField(...),
        a!buttonWidget(...)
      }
    )
  }
)

/* CORRECT - Use formLayout */
a!formLayout(
  contentsWidth: "MEDIUM",
  contents: {a!textField(...)},
  buttons: a!buttonLayout(...)
)
```

### ❌ Too Many Panes
```sail
/* WRONG - Maximum is 3 panes */
a!paneLayout(
  panes: {
    a!pane(...),
    a!pane(...),
    a!pane(...),
    a!pane(...)  /* TOO MANY */
  }
)
```

## Validation Checklist
- [ ] At least one pane has `width: "AUTO"` for fluid layout
- [ ] No nested headerContentLayouts or formLayouts (they can only be top-level)
- [ ] Maximum of 3 panes used
- [ ] Background colors set for each pane
- [ ] Pane widths are appropriate for their content

## Quick Decision Guide

**"Do I need multiple vertical sections that scroll independently?"**
- Yes → **PaneLayout**

**"Is this an application-style interface with distinct panels?"**
- Yes → **PaneLayout**

**"Does my interface need to fill exactly the full browser height?"**
- Yes → **PaneLayout**

**"Am I building a simple form or content page?"**
- Yes → **FormLayout** or **HeaderContentLayout**

Remember: PaneLayout is powerful but should only be used when you truly need full-height, multi-panel interfaces with independent scrolling. For simpler layouts, formLayout or headerContentLayout are more appropriate choices.