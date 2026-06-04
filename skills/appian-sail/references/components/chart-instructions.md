# SAIL Chart Components Usage Instructions

## Overview
SAIL provides chart components for visualizing data including area, bar, column, line, pie, and scatter charts. Charts support configuration for styling, data series, reference lines, and custom colors.

## ⚠️ CRITICAL: Two Different Data Approaches

### Approach 1: Static Mockup Data (categories + series)
**Use for:** Column, Line, Bar, Area, and Pie charts with hardcoded sample data
**Structure:**
```sail
a!columnChartField(
  categories: {"Q1", "Q2", "Q3", "Q4"},
  series: {
    a!chartSeries(label: "Sales", data: {100, 120, 115, 140}, color: "#3B82F6")
  }
)
```

### Approach 2: Record/Query Data (data + config)
**Use for:** All charts when connecting to live record data
**Structure:**
```sail
a!columnChartField(
  data: a!recordData(...),
  config: a!columnChartConfig(
    primaryGrouping: a!grouping(...),
    measures: {a!measure(...)}
  )
)
```

### Important Notes:
- ✅ For static mockups, ALWAYS use `categories` + `series`
- ❌ Do NOT use `data` + `config` for static mockups (will cause errors)
- ⚠️ **Scatter charts ONLY work with Approach 2 (record data)** - they cannot be used with static mockup data

## Available Chart Types

### a!areaChartField()
Displays data as filled areas under lines, useful for showing trends and cumulative values over time.

### a!barChartField()
Displays horizontal bars, ideal for comparing discrete categories or showing rankings.

### a!columnChartField()
Displays vertical bars, best for comparing values across categories or time periods.

### a!lineChartField()
Displays data as connected points, excellent for showing trends and changes over time.

### a!pieChartField()
Displays data as pie slices showing part-to-whole relationships.

### a!scatterChartField()
Displays data as points on X/Y axes, useful for showing correlations and distributions.

**⚠️ IMPORTANT: Scatter charts require record type data and CANNOT be used with static mockup data. They only accept the `data` parameter with a record type reference or `a!recordData()`, not the `categories` + `series` approach used by other charts.**

## When to Use Each Chart Type

### Area Chart
- **Best for**: Cumulative data, stacked values, showing volume over time
- **Examples**: Revenue accumulation, resource utilization, inventory levels

### Bar Chart
- **Best for**: Comparing many categories, showing rankings, horizontal comparisons
- **Examples**: Sales by region, product comparisons, survey results

### Column Chart
- **Best for**: Comparing values across categories, time-based comparisons
- **Examples**: Monthly sales, quarterly performance, year-over-year comparisons

### Line Chart
- **Best for**: Trends over time, continuous data, multiple series comparisons
- **Examples**: Stock prices, temperature changes, website traffic

### Pie Chart
- **Best for**: Part-to-whole relationships, percentage distributions (5-7 slices max)
- **Examples**: Market share, budget allocation, demographic breakdowns

### Scatter Chart
- **Best for**: Correlations, distributions, outlier identification
- **Examples**: Price vs. quality, age vs. income, performance metrics
- **⚠️ Limitation**: Requires record type data - cannot be used with static mockups

## Common Chart Parameters

### Parameters Shared by All Charts
- **label**: Chart field label (usually excluded in favor of a section heading)
- **labelPosition**: "ABOVE" (default), "COLLAPSED", "ADJACENT" (usually "COLLAPSED")
- **instructions**: Explanatory text shown below label
- **height**: Available values vary by chart type (see below)
- **showWhen**: Conditional display logic
- **accessibilityText**: Screen reader description
- **xAxisTitle**: Horizontal axis label (not available for pie charts)
- **yAxisTitle**: Vertical axis label (not available for pie charts)
- **showLegend**: Boolean - show/hide legend (column, line, bar, area only - not pie)
- **showDataLabels**: Boolean - show value labels on data points
- **colorScheme**: Predefined color scheme name (e.g., "RAINFOREST", "OCEAN")
- **stacking**: Stacking mode for column/bar/area charts - "NONE", "NORMAL", "PERCENT_TO_TOTAL"

### Height Options
Available height values vary by chart type:
- **Column, Line, Area Charts**: `MICRO`, `SHORT`, `MEDIUM`, `TALL`
- **Scatter, Pie Charts**: `SHORT`, `MEDIUM`, `TALL`
- **Bar Charts**: `MICRO`, `SHORT`, `MEDIUM`, `TALL`, `AUTO`

### Data Parameters for Mock Data (Column, Line, Bar, Area, Pie)
- **categories**: Array of text labels for the axis (e.g., `{"Q1", "Q2", "Q3"}`)
  - Required when using static data
  - One label for each data point
- **series**: Array of `a!chartSeries()` containing the actual data
  - Each series needs: `label`, `data` (array of numbers), `color` (hex or semantic)
  - For pie charts, each series is one slice with a single data value

### Data Parameters for Live Record Data
- **data**: Record type reference (e.g., `recordType!Product`) or `a!recordData()`
  - Required when connecting to live record data
  - Scatter charts ONLY support this approach
- **config**: Chart-specific configuration object with grouping and measures
  - `a!columnChartConfig()`, `a!lineChartConfig()`, etc.
  - Contains `primaryGrouping`, `measures`, `referenceLines`, etc.

## Chart Configuration Components

### a!chartSeries() - For Static Mockup Charts
Defines a data series within a chart. **Used with `categories` + `series` approach.**

**Parameters:**
- **label**: Series name (shown in legend)
- **data**: Array of numeric values (for static charts) OR single numeric value (for pie chart slices)
- **color**: Color for series - hex code (e.g., "#3B82F6") or semantic color ("ACCENT", "POSITIVE", "NEGATIVE", "SECONDARY")

**Example for column/line/bar/area charts:**
```sail
a!chartSeries(
  label: "Revenue",
  data: {50000, 62000, 58000, 71000},
  color: "#3B82F6"
)
```

**Example for pie charts:**
```sail
a!chartSeries(
  label: "Company A",
  data: 32,  /* Single value for pie slice */
  color: "#3B82F6"
)
```

---

## Configuration Components for Record-Based Charts ONLY

**⚠️ The following configuration components are ONLY used when connecting charts to record data using the `data` + `config` approach. Do NOT use these for static mockup charts.**

### Area Chart Config: a!areaChartConfig()
```sail
a!areaChartConfig(
  primaryGrouping: a!grouping(/* grouping config */),
  secondaryGrouping: a!grouping(/* optional secondary grouping */),
  measures: {a!measure(/* measure config */)},
  dataLimit: 100,
  series: {a!chartSeries(/* series config */)},
  xAxisTitle: "Time Period",
  yAxisTitle: "Value",
  yAxisMin: 0,
  yAxisMax: 100000,
  referenceLines: {a!chartReferenceLine(/* reference line config */)},
  colorScheme: a!chartCustomColorScheme(/* custom colors */),
  showLegend: true,
  showDataLabels: false,
  showTooltips: true,
  allowDecimalAxisLabels: false,
  stacking: "NORMAL" /* "NORMAL", "PERCENT", or "NONE" */
)
```

### Bar Chart Config: a!barChartConfig()
```sail
a!barChartConfig(
  primaryGrouping: a!grouping(/* grouping config */),
  secondaryGrouping: a!grouping(/* optional secondary grouping */),
  measures: {a!measure(/* measure config */)},
  dataLimit: 100,
  series: {a!chartSeries(/* series config */)},
  xAxisTitle: "Value",
  yAxisTitle: "Category",
  xAxisMin: 0,
  xAxisMax: 100,
  referenceLines: {a!chartReferenceLine(/* reference line config */)},
  colorScheme: a!chartCustomColorScheme(/* custom colors */),
  showLegend: true,
  showDataLabels: false,
  showTooltips: true,
  allowDecimalAxisLabels: false,
  stacking: "NONE" /* "NORMAL", "PERCENT", or "NONE" */
)
```

### Column Chart Config: a!columnChartConfig()
```sail
a!columnChartConfig(
  primaryGrouping: a!grouping(/* grouping config */),
  secondaryGrouping: a!grouping(/* optional secondary grouping */),
  measures: {a!measure(/* measure config */)},
  dataLimit: 100,
  series: {a!chartSeries(/* series config */)},
  xAxisTitle: "Category",
  yAxisTitle: "Value",
  yAxisMin: 0,
  yAxisMax: 100000,
  referenceLines: {a!chartReferenceLine(/* reference line config */)},
  colorScheme: a!chartCustomColorScheme(/* custom colors */),
  showLegend: true,
  showDataLabels: false,
  showTooltips: true,
  allowDecimalAxisLabels: false,
  stacking: "NONE" /* "NORMAL", "PERCENT", or "NONE" */
)
```

### Line Chart Config: a!lineChartConfig()
```sail
a!lineChartConfig(
  primaryGrouping: a!grouping(/* grouping config */),
  secondaryGrouping: a!grouping(/* optional secondary grouping */),
  measures: {a!measure(/* measure config */)},
  dataLimit: 100,
  series: {a!chartSeries(/* series config */)},
  xAxisTitle: "Time Period",
  yAxisTitle: "Value",
  yAxisMin: 0,
  yAxisMax: 100000,
  referenceLines: {a!chartReferenceLine(/* reference line config */)},
  colorScheme: a!chartCustomColorScheme(/* custom colors */),
  showLegend: true,
  showDataLabels: false,
  showTooltips: true,
  allowDecimalAxisLabels: false,
  connectNulls: true /* Whether to connect points across null values */
)
```

### Pie Chart Config: a!pieChartConfig()
```sail
a!pieChartConfig(
  primaryGrouping: a!grouping(/* grouping config */),
  measures: {a!measure(/* measure config */)},
  dataLimit: 10,
  series: {a!chartSeries(/* series config */)},
  colorScheme: a!chartCustomColorScheme(/* custom colors */),
  showLegend: true,
  showDataLabels: true,
  showTooltips: true,
  showAsDonut: false, /* true for donut chart */
  seriesLabelStyle: "ON_CHART" /* "ON_CHART", "IN_LEGEND", or "NONE" */
)
```

### a!grouping()
Defines how to group data for chart display.

```sail
a!grouping(
  field: "category",           /* Field name from data */
  alias: "Category",           /* Display name */
  intervals: null              /* Optional interval grouping */
)
```

### ⚠️ CRITICAL: Grouping Field Selection (Primary & Secondary)

Group by human-readable values (Text, Date, DateTime, Boolean), **never by numeric IDs** (Integer, Decimal).

This applies to BOTH `primaryGrouping` AND `secondaryGrouping`.

**Decision Logic:**
1. Base record has Text field? → Use directly
2. Base record has Date/DateTime? → Use with `interval` parameter
3. Base record only has Integer FK? → Navigate to related record's Text field

```sail
/* ❌ WRONG - grouping by Integer ID */
primaryGrouping: a!grouping(field: 'recordType!Case.fields.statusId', ...)

/* ✅ CORRECT - Text field in base record */
primaryGrouping: a!grouping(field: 'recordType!Case.fields.statusName', ...)

/* ✅ CORRECT - Related Text field when base record only has FK */
primaryGrouping: a!grouping(field: 'recordType!Case.relationships.status.fields.statusName', ...)

/* ✅ CORRECT - Date with interval */
primaryGrouping: a!grouping(
  field: 'recordType!Case.fields.submissionDate',
  interval: "DATE_TEXT"
)
```

**Quick Reference:**
| Field Type | Pattern |
|------------|---------|
| Text (in base record) | `...fields.textField` directly |
| Integer FK (no text in base) | `...relationships.X.fields.firstTextField` |
| Date (group by date) | `...fields.dateField` with `interval: "DATE_TEXT"` |
| Date (group by month) | `...fields.dateField` with `interval: "MONTH_SHORT_TEXT"` |
| Date (group by year) | `...fields.dateField` with `interval: "YEAR"` |
| Boolean | `...fields.booleanField` directly |

### a!measure()
Defines aggregated values to display.

```sail
a!measure(
  field: "amount",             /* Field name from data */
  function: "SUM",             /* "SUM", "AVG", "COUNT", "MIN", "MAX" */
  alias: "Total Amount",       /* Display name */
  formatValue: "DOLLAR"        /* "DECIMAL", "DOLLAR", "PERCENT", "NONE" */
)
```

### a!chartReferenceLine()
Adds a reference line to the chart.

```sail
a!chartReferenceLine(
  label: "Target",
  value: 75000,
  color: "#10B981",            /* Hex color */
  style: "DASHED"              /* "SOLID" or "DASHED" */
)
```

### a!chartCustomColorScheme() - For Record-Based Charts Only
Defines custom colors for chart series when using `data` + `config` approach.

**⚠️ For static mockup charts, specify colors directly in each `a!chartSeries()` instead.**

```sail
/* For record-based charts with config */
a!chartCustomColorScheme(
  colors: {"#3B82F6", "#10B981", "#F59E0B", "#EF4444"}
)
```

## Static Mockup Data Structure

### ✅ CORRECT: Simple Column/Bar Chart with Static Data
```sail
a!columnChartField(
  labelPosition: "COLLAPSED",
  categories: {"Electronics", "Furniture", "Clothing", "Food", "Books"},
  series: {
    a!chartSeries(
      label: "Sales",
      data: {45000, 32000, 28000, 51000, 18000},
      color: "#3B82F6"
    )
  },
  xAxisTitle: "Category",
  yAxisTitle: "Sales ($)",
  showDataLabels: true,
  height: "MEDIUM"
)
```

### ✅ CORRECT: Multi-Series Line/Area Chart with Static Data
```sail
a!lineChartField(
  labelPosition: "COLLAPSED",
  categories: {"Jan", "Feb", "Mar", "Apr", "May", "Jun"},
  series: {
    a!chartSeries(
      label: "Revenue",
      data: {85000, 92000, 88000, 105000, 98000, 112000},
      color: "#3B82F6"
    ),
    a!chartSeries(
      label: "Expenses",
      data: {52000, 48000, 51000, 55000, 49000, 53000},
      color: "#EF4444"
    )
  },
  xAxisTitle: "Month",
  yAxisTitle: "Amount ($)",
  showLegend: true,
  height: "TALL"
)
```

### ✅ CORRECT: Pie Chart with Static Data
```sail
a!pieChartField(
  labelPosition: "COLLAPSED",
  series: {
    a!chartSeries(label: "Company A", data: 32, color: "#3B82F6"),
    a!chartSeries(label: "Company B", data: 28, color: "#10B981"),
    a!chartSeries(label: "Company C", data: 18, color: "#F59E0B"),
    a!chartSeries(label: "Company D", data: 12, color: "#8B5CF6"),
    a!chartSeries(label: "Others", data: 10, color: "#EF4444")
  },
  style: "DONUT",
  seriesLabelStyle: "ON_CHART",
  height: "MEDIUM"
)
```

### ❌ WRONG: Do NOT use this approach for static mockups
```sail
/* This approach only works with record data, NOT static mockups */
a!columnChartField(
  data: local!salesData,  /* ❌ Will cause "Cannot index property 'uuid'" error */
  config: a!columnChartConfig(...)
)
```

## Chart Templates for Static Mockups

### Column Chart (Single Series)
```sail
a!columnChartField(
  labelPosition: "COLLAPSED",
  categories: {"Q1", "Q2", "Q3", "Q4"},
  series: {
    a!chartSeries(
      label: "Sales",
      data: {245000, 312000, 278000, 335000},
      color: "#3B82F6"
    )
  },
  xAxisTitle: "Quarter",
  yAxisTitle: "Sales Revenue ($)",
  showDataLabels: true,
  showLegend: false,
  height: "TALL"
)
```

### Line Chart (Multi-Series)
```sail
a!lineChartField(
  labelPosition: "COLLAPSED",
  categories: {"Week 1", "Week 2", "Week 3", "Week 4", "Week 5", "Week 6"},
  series: {
    a!chartSeries(
      label: "Unique Visitors",
      data: {12500, 13200, 14100, 13800, 15200, 16500},
      color: "#3B82F6"
    ),
    a!chartSeries(
      label: "Page Views",
      data: {45000, 48500, 52000, 50200, 55800, 60500},
      color: "#10B981"
    )
  },
  xAxisTitle: "Time Period",
  yAxisTitle: "Count",
  showLegend: true,
  height: "MEDIUM"
)
```

### Area Chart (Stacked)
```sail
a!areaChartField(
  labelPosition: "COLLAPSED",
  categories: {"Jan", "Feb", "Mar", "Apr", "May", "Jun"},
  series: {
    a!chartSeries(
      label: "Development",
      data: {15, 18, 16, 20, 19, 22},
      color: "#3B82F6"
    ),
    a!chartSeries(
      label: "Design",
      data: {8, 10, 9, 12, 11, 13},
      color: "#10B981"
    ),
    a!chartSeries(
      label: "Marketing",
      data: {5, 6, 7, 8, 9, 10},
      color: "#F59E0B"
    )
  },
  xAxisTitle: "Month",
  yAxisTitle: "Team Size",
  stacking: "NORMAL",
  showLegend: true,
  height: "TALL"
)
```

### Bar Chart (Horizontal)
```sail
a!barChartField(
  labelPosition: "COLLAPSED",
  categories: {"Product A", "Product B", "Product C", "Product D", "Product E"},
  series: {
    a!chartSeries(
      label: "Revenue",
      data: {485000, 412000, 378000, 325000, 298000},
      color: "#8B5CF6"
    )
  },
  xAxisTitle: "Revenue ($)",
  showDataLabels: true,
  showLegend: false,
  height: "TALL"
)
```

### Pie Chart (Donut Style)
```sail
a!pieChartField(
  labelPosition: "COLLAPSED",
  series: {
    a!chartSeries(label: "Personnel", data: 450000, color: "#3B82F6"),
    a!chartSeries(label: "Operations", data: 280000, color: "#10B981"),
    a!chartSeries(label: "Marketing", data: 180000, color: "#F59E0B"),
    a!chartSeries(label: "R&D", data: 220000, color: "#8B5CF6"),
    a!chartSeries(label: "Infrastructure", data: 120000, color: "#EF4444")
  },
  style: "DONUT",
  seriesLabelStyle: "ON_CHART",
  height: "MEDIUM"
)
```

### Scatter Chart
**⚠️ IMPORTANT: Scatter charts require record type data and cannot be demonstrated with static mockups.**

For scatter charts, you must use the `data` + `config` approach with a record type:
```sail
/* Scatter chart - ONLY works with record data */
a!scatterChartField(
  data: recordType!Product,  /* Must be a record type reference */
  config: a!scatterChartConfig(
    xAxisField: "quality",
    yAxisField: "price",
    xAxisTitle: "Quality Score",
    yAxisTitle: "Price ($)"
  ),
  height: "MEDIUM"
)
```

## Color Guidelines

### Valid Color Values
- **Semantic colors**: "ACCENT", "POSITIVE", "NEGATIVE", "SECONDARY"
- **Hex codes**: "#3B82F6", "#10B981", "#F59E0B", "#EF4444", etc. (always 6 characters)

### How to Apply Colors

**For Static Mockup Charts:**
Specify colors directly in each `a!chartSeries()`:
```sail
series: {
  a!chartSeries(label: "Revenue", data: {100, 150, 120}, color: "#3B82F6"),
  a!chartSeries(label: "Expenses", data: {80, 90, 85}, color: "#EF4444")
}
```

**For Record-Based Charts:**
Use `colorScheme` parameter with predefined scheme name OR `a!chartCustomColorScheme()`:
```sail
/* Option 1: Use predefined scheme */
colorScheme: "OCEAN"

/* Option 2: Use custom colors */
config: a!columnChartConfig(
  colorScheme: a!chartCustomColorScheme(
    colors: {"#3B82F6", "#10B981", "#F59E0B"}
  )
)
```

### Recommended Color Palettes

**Single series:**
- Primary: `"#3B82F6"` (Blue)
- Success: `"#10B981"` (Green)
- Warning: `"#F59E0B"` (Amber)
- Error: `"#EF4444"` (Red)

**Multi-series (balanced):**
```sail
{"#3B82F6", "#10B981", "#F59E0B", "#8B5CF6", "#EC4899"}
/* Blue, Green, Amber, Purple, Pink */
```

**Multi-series (professional/dark):**
```sail
{"#1E40AF", "#047857", "#92400E", "#5B21B6", "#831843"}
/* Dark blue, dark green, dark amber, dark purple, dark pink */
```

## Best Practices

### ✅ DO:
- Choose chart type appropriate for data and message
- Limit pie charts to 5-7 slices maximum
- Use consistent colors across related charts
- Provide clear axis titles and labels
- Include reference lines for targets or thresholds
- Set appropriate height for readability
- Use `showDataLabels: true` for small datasets (<10 points)
- Format values appropriately (dollar, percent, decimal)
- Include tooltips for detailed information

### ❌ DON'T:
- Use 3D or overly decorative styles
- Show too many series in one chart (3-5 max)
- Use similar colors that are hard to distinguish
- Omit axis titles or units
- Start Y-axis at arbitrary values (use 0 for most cases)
- Use pie charts for precise comparisons
- Make charts too short to read comfortably
- Use red/green only (consider colorblind users)

## Chart Data Requirements

### Data Structure Rules:
- Data must be an array of `a!map()` objects
- Each map must have consistent field names
- Field names in data must match `field` values in grouping/measures
- Numeric fields required for measures
- Text fields required for grouping

### Common Data Patterns:

**Category + Value:**
```sail
{a!map(category: "A", value: 100), a!map(category: "B", value: 150)}
```

**Time + Multiple Values:**
```sail
{a!map(date: "Jan", sales: 1000, costs: 600), a!map(date: "Feb", sales: 1200, costs: 650)}
```

**X + Y Coordinates (Scatter):**
```sail
{a!map(x: 10, y: 20), a!map(x: 15, y: 35)}
```

## Validation Checklist

Before finalizing any chart:

### For Static Mockup Charts:
- [ ] ✅ Using `categories` + `series` approach (NOT `data` + `config`)
- [ ] ✅ Each `a!chartSeries()` has `label`, `data` array, and `color`
- [ ] ✅ Number of data points matches number of categories
- [ ] ❌ NOT using scatter chart (requires record data)
- [ ] Chart type appropriate for data and message
- [ ] Colors use valid hex codes (6 characters) or semantic colors
- [ ] Axis titles provided where applicable
- [ ] Height set appropriately for content
- [ ] Legend parameter used correctly (not available for pie charts)
- [ ] Stacking parameter is valid: "NORMAL", "PERCENT_TO_TOTAL", or "NONE"
- [ ] All parameters match documented options for static charts

### For Record-Based Charts:
- [ ] Using `data` parameter with record type or `a!recordData()`
- [ ] Using `config` parameter with appropriate config function
- [ ] Grouping field names match fields in record type
- [ ] Measure field names match fields in record type
- [ ] All configuration parameters match documented options

## Common Mistakes to Avoid

### 🚨 MOST CRITICAL ERROR - Using Wrong Data Approach

**The #1 most common mistake is using `data` + `config` for static mockups.**

This causes the error: `"Cannot index property 'uuid' of type Text into type List of Null"`

**Fix:** Always use `categories` + `series` for static mockup charts!

---

### ❌ WRONG - Invalid stacking value:
```sail
stacking: "STACKED"  /* ❌ NOT VALID */
```

### ✅ CORRECT - Valid stacking values:
```sail
stacking: "NORMAL"   /* ✅ Regular stacking */
stacking: "PERCENT"  /* ✅ Percentage stacking */
stacking: "NONE"     /* ✅ No stacking */
```

### ❌ WRONG - Invalid color:
```sail
color: "RED"  /* ❌ HTML color names not valid */
```

### ✅ CORRECT - Valid colors:
```sail
color: "#EF4444"     /* ✅ Hex code */
color: "NEGATIVE"    /* ✅ Semantic color */
```

### ❌ WRONG - Using data + config for static mockups:
```sail
/* This will cause "Cannot index property 'uuid'" error */
a!columnChartField(
  data: {a!map(category: "A", value: 100)},
  config: a!columnChartConfig(
    primaryGrouping: a!grouping(field: "category"),
    measures: {a!measure(field: "value", function: "SUM")}
  )
)
```

### ✅ CORRECT - Using categories + series for static mockups:
```sail
a!columnChartField(
  categories: {"A", "B", "C"},
  series: {
    a!chartSeries(label: "Values", data: {100, 150, 120}, color: "#3B82F6")
  }
)
```
