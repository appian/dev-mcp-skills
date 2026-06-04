# SAIL ColumnsLayout Usage Instructions

## Overview
ColumnsLayout is the primary structural layout component in Appian SAIL for arranging content horizontally across the page. It should be used instead of sideBySideLayout for main page structure.

## Core Principles

### Column Width Types
1. **Fixed-width columns**: Use specific width enumerations for consistent content areas
2. **Fluid columns**: Use "AUTO" width to fill remaining horizontal space
3. **Margin columns**: Use "AUTO" width columns as flexible margins

### Width Enumerations
- `EXTRA_NARROW`: 80px
- `NARROW`: 240px  
- `NARROW_PLUS`: 320px
- `MEDIUM`: 400px
- `MEDIUM_PLUS`: 560px
- `WIDE`: 800px
- `WIDE_PLUS`: 1120px
- `EXTRA_WIDE`: 1440px
- `AUTO`: Fluid width (fills remaining space)

### Required Rules
- At least one column must have `width: "AUTO"` for fluid layout
- Use AUTO columns as margins for centered content layouts

### Spacing
- Determines the spacing (gutters) between columns:
  - `STANDARD` (default)
  - `NONE` (no space between columns)
  - `DENSE` (less space between columns)
  - `SPARSE` (more space between columns)

### Vertical Alignment
- Controls how content aligns vertically within columns of different heights:
  - `TOP` (default): Align content to top of column
  - `MIDDLE`: Center content vertically - useful for aligning icons, stamps, and text across columns
  - `BOTTOM`: Align content to bottom of column

## Page Layout Strategy Using Columns Layout

### Option A: Content Columns Using Full Page Width
Content columns distribute the width of the page without extra margins
```sail
a!columnsLayout(
  columns: {
    a!columnLayout(width: "AUTO", contents: { /* Main content */ }),
    a!columnLayout(width: "MEDIUM", contents: { /* Sidebar content */ })
  }
)
```

### Option B: Constrained Width (Centered Content Within Maximum Width)
All content in constrained-width center column with adaptive margins 
⚠️⚠️⚠️ ONLY use if contents can be confined to max width of 1440px
```sail
a!columnsLayout(
  columns: {
    a!columnLayout(width: "AUTO", showWhen: a!isPageWidth({"DESKTOP","DESKTOP_WIDE"})), /* left margin column, hide on narrow screens */
    a!columnLayout( /* fixed-width column for page contents */
      width: if(a!isPageWidth({"DESKTOP","DESKTOP_WIDE"}), "EXTRA_WIDE", "AUTO"),
      contents: { /* ALL page content goes here */ }
    ),
    a!columnLayout(width: "AUTO", showWhen: a!isPageWidth({"DESKTOP","DESKTOP_WIDE"})) /* right margin column, hide on narrow screens */
  }
)
```

### Option C: Hybrid (Fixed + Fluid with Optional Margins)
Mix of fixed-width and fluid columns with adaptive margins
```sail
a!columnsLayout(
  columns: {
    a!columnLayout(width: "AUTO", showWhen: a!isPageWidth({"DESKTOP_WIDE"})), /* left margin column, hide on narrow screens */
    a!columnLayout(
      width: if(a!isPageWidth({"DESKTOP_WIDE"}), "WIDE", "AUTO"),
      contents: { /* Main content - switches to AUTO when margins hidden */ }
    ),
    a!columnLayout(width: "MEDIUM", contents: { /* Fixed sidebar */ }),
    a!columnLayout(width: "AUTO", showWhen: a!isPageWidth({"DESKTOP_WIDE"})) /* right margin column, hide on narrow screens */
  }
)
```

### Option D: No Columns Layout Needed - Content in Single Stack Fills Full Width of Page
Single stack of content - put directly in top-level layout (headerContentLayout, formLayout, etc.)
```sail
/* Put content directly in headerContentLayout contents, formLayout contents, etc. */
a!headerContentLayout(
  contents: {
    /* All page content in vertical stack */
    a!cardLayout(...),
    a!sectionLayout(...),
    a!gridField(...)
  }
)
```

## Converting from Tailwind CSS to SAIL Columns

### Tailwind Grid to SAIL Columns
```css
/* Tailwind CSS */
.grid-cols-12 {
  grid-template-columns: repeat(12, minmax(0, 1fr));
}
.col-span-4 { grid-column: span 4 / span 4; }
.col-span-8 { grid-column: span 8 / span 8; }
```

**Converts to SAIL:**
```sail
a!columnsLayout(
  columns: {
    a!columnLayout(width: "MEDIUM", contents: {...}),      /* ~33% */
    a!columnLayout(width: "AUTO", contents: {...})         /* ~67% */
  }
)
```

### Tailwind Container with Max-Width
```css
/* Tailwind CSS */
.container .mx-auto .max-w-4xl
```

**Converts to SAIL:**
```sail
a!columnsLayout(
  columns: {
    a!columnLayout(width: "AUTO"),                     /* Left margin */
    a!columnLayout(width: "EXTRA_WIDE", contents: {...}), /* Max-width content */
    a!columnLayout(width: "AUTO")                      /* Right margin */
  }
)
```

## Width Selection Guide

### Content Type → Recommended Width
- **Navigation sidebar**: `MEDIUM` or `MEDIUM_PLUS`
- **Main content area**: `WIDE_PLUS` or `EXTRA_WIDE` (for wide-screen dashboards and data-heavy layouts)
- **Secondary content**: `MEDIUM` or `MEDIUM_PLUS`
- **Flexible content**: `AUTO`
- **Margins/spacing**: `AUTO`

## Best Practices

### ✅ DO:
- Use `AUTO` columns for margins in centered layouts
- Choose fixed widths based on content requirements
- Use columnsLayout for main page structure
- Set appropriate `alignVertical`

### ❌ DON'T:
- Use all fixed-width columns (responsive issues)
- Use sideBySideLayout for main page structure
- Nest columnsLayout inside sideBySideLayout
- Forget margin columns for centered content

## Responsive Behavior
- Fixed-width columns maintain their pixel widths
- AUTO columns shrink/expand to accommodate screen size
- On narrow screens, columns may stack vertically (controlled by `stackWhen` parameter)

## MANDATORY VALIDATION CHECKLIST
- [ ] At least one column has `AUTO` width
- [ ] Column width is `AUTO`, `EXTRA_NARROW`, `NARROW`, `NARROW_PLUS`, `MEDIUM`, `MEDIUM_PLUS`, `WIDE`, `WIDE_PLUS`, `EXTRA_WIDE`, `1X`, `2X`, `3X`, `4X`, `5X`, `6X`, `7X`, `8X`, `9X`, or `10X`
- [ ] ‼️ MAKE SURE that you're ONLY using one of the VALID VALUES for `spacing` and `width`
- [ ] `alignVertical` is set unless default of `TOP` is desired
- [ ] ⚠️⚠️⚠️ `spacing` is `STANDARD`, `NONE`, `SPARSE`, or `DENSE`
- [ ] ❌ DON'T USE `LESS` or `MORE` for `spacing`!
- [ ] ⚠️⚠️⚠️ `spacing` is `STANDARD`, `NONE`, `SPARSE`, or `DENSE`
- [ ] ❌ DON'T USE `LESS` or `MORE` for `spacing`!
- [ ] ⚠️⚠️⚠️ `spacing` is `STANDARD`, `NONE`, `SPARSE`, or `DENSE`
- [ ] ❌ DON'T USE `LESS` or `MORE` for `spacing`!