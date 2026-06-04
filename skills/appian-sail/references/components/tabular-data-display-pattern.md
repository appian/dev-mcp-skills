# Rich Custom Tabular Data Display Pattern

## Overview
The Custom Tabular Data Display pattern provides an alternative to `a!gridField` for complex visual representations of tabular data. This pattern uses `a!cardLayout`, `a!columnsLayout`, and `a!forEach` to create table-like structures that support rich content combinations within cells, including nested layouts and multiple components per cell.

## ⚠️ CRITICAL DECISION POINT: GridField vs Custom Tabular Pattern

### Use GridField When:
- Cells contain **single components only** (text, tags, buttons, rich text, progress bars)
- Data is relatively simple and doesn't require complex layouts within cells

### Use Custom Tabular Pattern When:
- Cells need **multiple component types** (stamp + text + tag in same cell)
- Cells need **nested layouts** (sideBySideLayout, columnsLayout within cells)
- Cells need **stacked components** (rich text + progress bar, multiple lines of content)
- Custom table styling is required beyond gridField capabilities
- Table needs non-standard visual design or branding

## Quick Identification Guide

**Use Custom Pattern for these cell types:**
- ✋ **Multiple components in one cell** (e.g., avatar + name + status badge)
- ✋ **Nested layouts within cells** (e.g., sideBySideLayout with icon + text + tag)
- ✋ **Stacked content in cells** (e.g., title over subtitle over progress bar)
- ✋ **Custom cell backgrounds** or complex visual styling
- ✋ **Non-standard table structure** (merged cells, custom headers)

**GridField is sufficient when:**
- ✅ **Single component per cell** (one rich text, one tag, one button)
- ✅ **Simple data display** without complex visual requirements

## Core Pattern Structure

### Complete Example within Optional Section Card
```sail
a!localVariables(
  local!tableData: {
    a!map(
      name: "TechSolutions Inc.",
      logo: "TS",
      overallScore: 85,
      totalCost: "$2.4M",
      status: "compliant",
      riskLevel: "low"
    ),
    a!map(
      name: "Global Systems Corp",
      logo: "GS", 
      overallScore: 78,
      totalCost: "$2.1M",
      status: "pending",
      riskLevel: "medium"
    )
  },
  
  
  a!cardLayout( /* Optional: Section Card Wrapper */
    contents: {
      /* Section Title */
      a!cardLayout(
        contents: {
          a!headingField(
            text: "Vendor Comparison Matrix",
            size: "MEDIUM_PLUS",
            headingTag: "H2",
            fontWeight: "SEMI_BOLD",
            marginBelow: "EVEN_LESS"
          ),
          a!richTextDisplayField(
            value: a!richTextItem(
              text: "Click on any data point to view supporting documentation",
              size: "SMALL",
              color: "STANDARD"
            ),
            labelPosition: "COLLAPSED"
          )
        },
        style: "#FFFFFF",
        showBorder: false,
        padding: "MORE",
        marginBelow: "NONE"
      ),
      
      /* Top Border */
      a!horizontalLine(
        color: "SECONDARY",
        weight: "THIN",
        marginBelow: "NONE"
      ),
      
      /* Column Headers */
      a!cardLayout(
        contents: {
          a!columnsLayout(
            columns: {
              a!columnLayout(
                contents: a!richTextDisplayField(
                  value: a!richTextItem(
                    text: "VENDOR",
                    size: "SMALL",
                    style: "STRONG",
                    color: "STANDARD"
                  ),
                  labelPosition: "COLLAPSED"
                ),
                width: "MEDIUM"
              ),
              a!columnLayout(
                contents: a!richTextDisplayField(
                  value: a!richTextItem(
                    text: "OVERALL SCORE",
                    size: "SMALL",
                    style: "STRONG",
                    color: "STANDARD"
                  ),
                  align: "CENTER",
                  labelPosition: "COLLAPSED"
                ),
                width: "NARROW"
              ),
              a!columnLayout(
                contents: a!richTextDisplayField(
                  value: a!richTextItem(
                    text: "TOTAL COST",
                    size: "SMALL",
                    style: "STRONG",
                    color: "STANDARD"
                  ),
                  align: "CENTER",
                  labelPosition: "COLLAPSED"
                ),
                width: "NARROW"
              ),
              a!columnLayout(
                contents: a!richTextDisplayField(
                  value: a!richTextItem(
                    text: "ACTIONS",
                    size: "SMALL",
                    style: "STRONG",
                    color: "STANDARD"
                  ),
                  align: "CENTER",
                  labelPosition: "COLLAPSED"
                ),
                width: "NARROW"
              )
            },
            spacing: "STANDARD"
          )
        },
        style: "#FFFFFF",
        showBorder: false,
        padding: "MORE",
        marginBelow: "NONE"
      ),
      
      /* Header Bottom Border */
      a!horizontalLine(
        color: "SECONDARY",
        weight: "THIN",
        marginBelow: "NONE"
      ),
      
      /* Data Rows */
      a!forEach(
        items: local!tableData,
        expression: {
          a!cardLayout(
            contents: {
              a!columnsLayout(
                columns: {
                  /* Complex Cell: Stamp + Text + Tag */
                  a!columnLayout(
                    contents: a!sideBySideLayout(
                      items: {
                        a!sideBySideItem(
                          item: a!stampField(
                            text: fv!item.logo,
                            backgroundColor: "ACCENT",
                            size: "TINY",
                            shape: "ROUNDED",
                            labelPosition: "COLLAPSED"
                          ),
                          width: "MINIMIZE"
                        ),
                        a!sideBySideItem(
                          item: a!richTextDisplayField(
                            value: a!richTextItem(
                              text: fv!item.name,
                              size: "STANDARD",
                              style: "STRONG"
                            ),
                            labelPosition: "COLLAPSED"
                          ),
                          width: "MINIMIZE"
                        ),
                        a!sideBySideItem(
                          item: a!tagField(
                            tags: a!tagItem(
                              text: upper(fv!item.status),
                              backgroundColor: if(
                                fv!item.status = "compliant",
                                "POSITIVE",
                                "SECONDARY"
                              )
                            ),
                            size: "SMALL",
                            labelPosition: "COLLAPSED"
                          ),
                          width: "AUTO"
                        )
                      },
                      alignVertical: "MIDDLE",
                      spacing: "STANDARD"
                    ),
                    width: "MEDIUM"
                  ),
                  
                  /* Stacked Cell: Score + Progress Bar */
                  a!columnLayout(
                    contents: {
                      a!richTextDisplayField(
                        value: a!richTextItem(
                          text: fv!item.overallScore,
                          size: "LARGE",
                          style: "STRONG",
                          color: "POSITIVE"
                        ),
                        align: "CENTER",
                        labelPosition: "COLLAPSED",
                        marginBelow: "EVEN_LESS"
                      ),
                      a!progressBarField(
                        percentage: fv!item.overallScore,
                        color: "ACCENT",
                        style: "THIN",
                        showPercentage: false,
                        labelPosition: "COLLAPSED"
                      )
                    },
                    width: "NARROW"
                  ),
                  
                  /* Multi-line Cell */
                  a!columnLayout(
                    contents: a!richTextDisplayField(
                      value: {
                        a!richTextItem(
                          text: fv!item.totalCost,
                          size: "MEDIUM",
                          style: "STRONG"
                        ),
                        char(10),
                        a!richTextItem(
                          text: "36 months",
                          size: "SMALL",
                          color: "SECONDARY"
                        )
                      },
                      align: "CENTER",
                      labelPosition: "COLLAPSED"
                    ),
                    width: "NARROW"
                  ),
                  
                  /* Actions Cell */
                  a!columnLayout(
                    contents: a!buttonArrayLayout(
                      buttons: {
                        a!buttonWidget(
                          icon: "eye",
                          label: "View",
                          style: "OUTLINE",
                          color: "SECONDARY",
                          size: "SMALL"
                        ),
                        a!buttonWidget(
                          icon: "trash",
                          style: "OUTLINE",
                          color: "SECONDARY",
                          size: "SMALL"
                        )
                      },
                      marginBelow: "NONE",
                      align: "CENTER"
                    ),
                    width: "NARROW"
                  )
                },
                spacing: "STANDARD",
                alignVertical: "MIDDLE"
              )
            },
            style: "#FFFFFF",
            showBorder: false,
            padding: "MORE",
            marginBelow: "NONE"
          ),
          
          /* Row Border (except last row) */
          if(
            not(fv!isLast),
            a!horizontalLine(
              color: "SECONDARY",
              weight: "THIN",
              marginBelow: "NONE"
            ),
            {}
          )
        }
      )
    },
    style: "#FFFFFF",
    shape: "ROUNDED",
    showBorder: true,
    padding: "NONE"                /* No padding on outer card to allow table borders to be flush with edge of card */
  )
)
```

### Basic Table Structure (Without Section Card)
```sail
a!localVariables(
  local!tableData: {
    /* Array of a!map() items */
  },
  
  /* Table without section wrapper - just the table itself */
  {
    /* Column Headers */
    a!cardLayout(
      contents: {
        a!columnsLayout(
          columns: { /* Header columns */ },
          spacing: "STANDARD"
        )
      },
      style: "#FFFFFF",
      showBorder: false,
      padding: "MORE"
    ),
    
    /* Header Bottom Border */
    a!horizontalLine(color: "SECONDARY", weight: "THIN"),
    
    /* Data Rows */
    a!forEach(
      items: local!tableData,
      expression: {
        a!cardLayout(
          contents: {
            a!columnsLayout(
              columns: { /* Data columns */ },
              spacing: "STANDARD",
              alignVertical: "MIDDLE"
            )
          },
          style: "#FFFFFF",
          showBorder: false,
          padding: "MORE"
        ),
        
        /* Row Border (except last row) */
        if(
          not(fv!isLast),
          a!horizontalLine(color: "SECONDARY", weight: "THIN"),
          {}
        )
      }
    )
  }
)
```

## Section Card Wrapper (Optional)

The outer section card wrapper is **optional** and should be used when:
- The table needs consistent styling with other card-based content sections on the page
- The table needs a title or description

### When to Use Section Card Wrapper:
✅ **Use when:**
- Table needs a title or subtitle
- Table is part of a dashboard with other cards
- Visual consistency with card-based layouts is important
- Table needs rounded corners and shadows

❌ **Skip when:**
- Table is the only content on the page
- Table should blend with page background
- Minimal visual styling is preferred
- Table is embedded within other content

## Column Structure Guidelines
- **Match widths**: Column headers and data columns must use identical width values
- **Consistent spacing**: Use the same `spacing` parameter for all `columnsLayout` instances
- **Consistent padding**: Use the same `padding` parameter for all `cardLayout` instances wrapping the header/rows
- **Vertical alignment**: Set `alignVertical: "MIDDLE"` for consistent row appearance

## Complex Cell Content Patterns

### 1. Stamp + Text + Tag Cell
```sail
a!columnLayout(
  contents: a!sideBySideLayout(
    items: {
      a!sideBySideItem(
        item: a!stampField(
          text: fv!item.initials,
          backgroundColor: "ACCENT",
          size: "TINY",
          shape: "ROUNDED",
          labelPosition: "COLLAPSED"
        ),
        width: "MINIMIZE"
      ),
      a!sideBySideItem(
        item: a!richTextDisplayField(
          value: a!richTextItem(
            text: fv!item.name,
            style: "STRONG"
          ),
          labelPosition: "COLLAPSED"
        ),
        width: "MINIMIZE"
      ),
      a!sideBySideItem(
        item: a!tagField(
          tags: a!tagItem(
            text: fv!item.status,
            backgroundColor: "POSITIVE"
          ),
          size: "SMALL",
          labelPosition: "COLLAPSED"
        ),
        width: "AUTO"
      )
    },
    alignVertical: "MIDDLE",
    spacing: "STANDARD"
  ),
  width: "MEDIUM"
)
```

### 2. Stacked Content Cell
```sail
a!columnLayout(
  contents: {
    a!richTextDisplayField(
      value: a!richTextItem(
        text: fv!item.title,
        style: "STRONG",
        size: "MEDIUM"
      ),
      align: "CENTER",
      labelPosition: "COLLAPSED",
      marginBelow: "EVEN_LESS"
    ),
    a!progressBarField(
      percentage: fv!item.completion,
      color: "ACCENT",
      style: "THIN",
      showPercentage: false,
      labelPosition: "COLLAPSED"
    )
  },
  width: "NARROW"
)
```

### 3. Multi-Line Rich Text Cell
Can also be used in gridField
```sail
a!columnLayout(
  contents: a!richTextDisplayField(
    value: {
      a!richTextItem(
        text: fv!item.primaryText,
        style: "STRONG",
        size: "STANDARD"
      ),
      char(10),
      a!richTextItem(
        text: fv!item.secondaryText,
        color: "SECONDARY",
        size: "SMALL"
      )
    },
    align: "CENTER",
    labelPosition: "COLLAPSED"
  ),
  width: "NARROW"
)
```

### 4. Action Buttons Cell
Can also be used in gridField
```sail
a!columnLayout(
  contents: a!buttonArrayLayout(
    buttons: {
      a!buttonWidget(
        icon: "eye",
        label: "View",
        style: "OUTLINE",
        color: "SECONDARY",
        size: "SMALL"
      ),
      a!buttonWidget(
        icon: "trash",
        style: "OUTLINE",
        color: "SECONDARY",
        size: "SMALL"
      )
    },
    marginBelow: "NONE",
    align: "CENTER"
  ),
  width: "NARROW"
)
```

## Table Header Pattern

### Simple Text Headers
```sail
a!cardLayout(
  contents: {
    a!columnsLayout(
      columns: {
        a!columnLayout(
          contents: a!richTextDisplayField(
            value: a!richTextItem(
              text: "COLUMN NAME",
              size: "SMALL",
              style: "STRONG",
              color: "STANDARD"
            ),
            labelPosition: "COLLAPSED"
          ),
          width: "MEDIUM"
        )
        /* Repeat for each column */
      },
      spacing: "STANDARD"
    )
  },
  style: "#FFFFFF",
  showBorder: false,
  padding: "MORE"                /* ✅ Inner card has padding */
)
```

## Data Source Pattern

### Using a!map for Static Data
```sail
a!localVariables(
  local!tableData: {
    a!map(
      id: "001",
      name: "John Doe",
      initials: "JD",
      status: "Active",
      score: 85,
      cost: "$2,400",
      riskLevel: "low"
    ),
    a!map(
      id: "002", 
      name: "Jane Smith",
      initials: "JS",
      status: "Pending",
      score: 72,
      cost: "$3,100",
      riskLevel: "medium"
    )
  },
  
  /* Table implementation */
)
```

## Styling and Visual Design

### Border and Background Styling
```sail
/* Outer table container (with section card) */
a!cardLayout(
  style: "#FFFFFF",           /* White background - match other cards on the page */
  shape: "ROUNDED",           /* Rounded corners - but match the rest of the page */
  showBorder: true,           /* Outer border */
  padding: "NONE"             /* ⚠️ CRITICAL: No padding for flush horizontalLines */
)

/* Individual content sections (title, header, rows) */
a!cardLayout(
  style: "#FFFFFF",           /* White row background */
  showBorder: false,          /* No individual borders */
  padding: "MORE"             /* ✅ Inner padding for content spacing */
)

/* Row separators */
a!horizontalLine(
  color: "SECONDARY",         /* Standard secondary color */
  weight: "THIN"
)
```

### Alternating Row Colors
```sail
a!forEach(
  items: local!tableData,
  expression: {
    a!cardLayout(
      contents: { /* Row content */ },
      style: if(
        mod(fv!index, 2) = 0,
        "#F9FAFB",              /* Light gray for even rows */
        "#FFFFFF"               /* White for odd rows */
      ),
      showBorder: false,
      padding: "MORE"             /* ✅ Each row card has padding */
    )
  }
)
```

## Validation Checklist

### Structure Validation:
- [ ] Column header widths match data column widths exactly
- [ ] All `columnsLayout` instances use same `spacing` value
- [ ] Row separators implemented with `horizontalLine` using `color: "SECONDARY"`
- [ ] **Outer container uses `padding: "NONE"` for flush borders**
- [ ] **Each content section wrapped in individual cards with `padding: "MORE"`**
- [ ] Inner cards have `showBorder: false` to prevent double borders

### Content Validation:
- [ ] Complex cell content uses appropriate nested layouts
- [ ] `labelPosition: "COLLAPSED"` set on all field components
- [ ] Consistent `alignVertical: "MIDDLE"` for row-level alignment
- [ ] Button arrays used instead of individual buttons
- [ ] Rich text used for formatted text content

### Padding Strategy Validation:
- [ ] Outer card wrapper has `padding: "NONE"`
- [ ] Title/header section wrapped in card with `padding: "MORE"`
- [ ] Each data row wrapped in card with `padding: "MORE"`
- [ ] Horizontal lines are flush with outer card edges
- [ ] No double borders or spacing issues

## Quick Reference: When to Use Each Approach

**"Do my table cells need multiple components or nested layouts?"**
- Yes → **Custom Tabular Pattern**
- No → **GridField**

**"Do I need custom visual styling beyond gridField capabilities?"**
- Yes → **Custom Tabular Pattern**
- No → **GridField**

Remember: GridField should be the default choice for tabular data. Only use the Custom Tabular Pattern when gridField's limitations prevent achieving the required design or functionality.