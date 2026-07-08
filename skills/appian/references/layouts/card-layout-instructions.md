# SAIL Card Layout Usage Instructions

## Overview
SAIL card layouts provide styled containers for content with borders, shadows, and various styling options. Use cards to group related content and create visual hierarchy.

## Core Card Components

### a!cardLayout
Primary component for individual cards containing content.

```sail
a!cardLayout(
    contents: { /* Contents */ },
    style: "#FFFFFF", 
    shape: "ROUNDED",
    height: "AUTO",
    showShadow: true,
    showBorder: false,
    padding: "STANDARD"
)
```

**Key Parameters:**
- `contents`: The content to display within the card
- `style`: Background color (NONE - white, TRANSPARENT, STANDARD - light gray, ACCENT, SUCCESS, INFO, WARN, ERROR, color schemes, or hex)
- `shape`: Corner rounding (SQUARED, SEMI_ROUNDED, ROUNDED)
- `padding`: Internal spacing (NONE, EVEN_LESS, LESS, STANDARD, MORE, EVEN_MORE) — default is `"LESS"`, not `"STANDARD"`
- `showBorder`: Whether to display outer border (default: true)
- `showShadow`: Whether to display drop shadow (default: false)
- `height`: Card height (EXTRA_SHORT through EXTRA_TALL, or AUTO)
- `link`: Makes entire card clickable
- `decorativeBarPosition`: Colored accent bar (TOP, BOTTOM, START, END, NONE)

### a!cardGroupLayout
Container for multiple cards with automatic flow and height matching.

**Key Parameters:**
- `cards`: Array of a!cardLayout components
- `cardWidth`: Consistent width for all cards (EXTRA_NARROW through EXTRA_WIDE)
- `cardHeight`: Consistent height for all cards (AUTO takes height of tallest card's contents)
- `spacing`: Space between cards (STANDARD, NONE, DENSE, SPARSE)
- `fillContainer`: Whether cards expand to fill available space

## Arranging Cards: columnsLayout vs cardGroupLayout

### When to Use cardGroupLayout
✅ **Best for:**
- Automatic card wrapping and responsive behavior
- When you want automatic height matching across all cards
- Quick implementation of card collections
- When the number of cards is known or unknown
- *DEFAULT* to using cardGroupLayout (not columnsLayout) to arrange sets of cards

### ✅ ALWAYS Use cardGroupLayout When:
- Displaying 2+ cards side-by-side that should have **equal heights**
- Cards contain variable-length content (descriptions, lists, text blocks)
- You want automatic height matching without manual height settings

**Key benefit:** `cardHeight: "AUTO"` automatically matches all cards to the tallest card's content height, creating a polished grid appearance.

**Example:**
```sail
a!cardGroupLayout(
  cards: {
    a!cardLayout(contents: {/* content */}, shape: "ROUNDED"),
    a!cardLayout(contents: {/* content */}, shape: "ROUNDED"),
    a!cardLayout(contents: {/* content */}, shape: "ROUNDED")
  },
  cardWidth: "NARROW_PLUS",
  spacing: "SPARSE",
  cardHeight: "AUTO"
)
```

### When to Use columnsLayout
✅ **Best for:**
- Specific grid arrangements that don't auto-wrap (like always a row of 3 cards)
- Cards that need different widths (but same widths is also fine)
- EVEN when above is true, if you need automatic height matching across cards, use cardGroupLayout instead!

**Example: Cards with same widths - always on one row**
```sail
a!columnsLayout(
  columns: {
    a!columnLayout(
      contents: a!cardLayout(contents: {/* content */}, shape: "ROUNDED"),
    ),
    a!columnLayout(
      contents: a!cardLayout(contents: {/* content */}, shape: "ROUNDED"), 
    ),
    a!columnLayout(
      contents: a!cardLayout(contents: {/* content */}, shape: "ROUNDED"),
    )
  }
)
```

**Example: Cards with different widths**
```sail
a!columnsLayout(
  columns: {
    a!columnLayout(
      contents: a!cardLayout(contents: {/* content */}, shape: "ROUNDED"),
      width: "WIDE"
    ),
    a!columnLayout(
      contents: a!cardLayout(contents: {/* content */}, shape: "ROUNDED"), 
      width: "MEDIUM"
    ),
    a!columnLayout(
      contents: a!cardLayout(contents: {/* content */}, shape: "ROUNDED"),
      width: "NARROW"
    )
  }
)
```

## Styling Decisions
- [ ] Choose the right card background color - set using the STYLE parameter
    - Use "NONE" or "#FFFFFF" for white cards
    - Use hex value for style to perfectly match card colors
    - Use "TRANSPARENT" and hide border for nested spacer cards
- [ ] Choose the right corner radius - set using the SHAPE parameter
    - "ROUNDED" is a good default for modern appearance
    - "SQUARED" to turn off border radius, such as for a headerContentLayout header card that's flush
- [ ] Should the card show a border? - set using the showBorder parameter
- [ ] Should the card show a drop shadow? - set using the showShadow parameter
- [ ] Choose the right amount of padding around card contents - set using the padding parameter
    - NONE: 0px
    - EVEN_LESS: 5px
    - LESS: 10px
    - STANDARD: 21px
    - MORE: 32px
    - EVEN_MORE: 64px
- [ ] Choose the right card height - set using the height parameter
    - AUTO (default): as tall as the card's contents - USE THIS AS MUCH AS POSSIBLE
    - EXTRA_SHORT: 60px
    - SHORT: 120px
    - SHORT_PLUS: 180px
    - MEDIUM: 240px
    - MEDIUM_PLUS: 300px
    - TALL: 360px
    - TALL_PLUS: 480px
    - EXTRA_TALL: 600px
- [ ] If using cardGroupLayout, choose the right card width - set using the cardWidth parameter:
    - EXTRA_NARROW: 80px
    - NARROW: 240px
    - NARROW_PLUS: 320px
    - MEDIUM: 400px
    - MEDIUM_PLUS: 560px
    - WIDE: 800px
    - WIDE_PLUS: 1120px

### Choosing cardWidth Based on Card Count

**Rule of thumb:** Assume ~1200-1400px viewport width for typical desktop layouts.

**For KPI cards (compact content with icon + metric):**
- **2 cards per row:** `MEDIUM` (400px) or `MEDIUM_PLUS` (560px)
- **3 cards per row:** `NARROW_PLUS` (320px) - fits comfortably
- **4 cards per row:** `NARROW` (240px) - fits on one row
- **5+ cards per row:** `EXTRA_NARROW` (80px) or consider pagination/scrolling

**For charts displayed side-by-side:**
- **2 charts:** `WIDE` (800px) - **recommended** for balanced layout with good chart readability
- **2 charts (compact):** `MEDIUM` (400px) - fits but leaves whitespace, charts may be too small
- **3 charts:** `NARROW_PLUS` (320px) - fits three across
- **4+ charts:** Consider stacking vertically or using tabs instead

**For content cards (lists, forms, detailed content):**
- **2 cards:** `WIDE` (800px) or `MEDIUM_PLUS` (560px)
- **3 cards:** `MEDIUM` (400px) or `NARROW_PLUS` (320px)
- **4 cards:** `NARROW_PLUS` (320px) or `NARROW` (240px)

**Quick calculation:** `Viewport width (~1200px) / cardWidth = cards per row`

Examples:
- `NARROW` (240px): 1200 / 240 = **~5 cards** per row
- `NARROW_PLUS` (320px): 1200 / 320 = **~3-4 cards** per row
- `MEDIUM` (400px): 1200 / 400 = **~3 cards** per row
- `WIDE` (800px): 1200 / 800 = **~1-2 cards** per row

**Common mistakes:**
```sail
/* ❌ WRONG - 4 KPIs with NARROW_PLUS wraps to 2 rows */
a!cardGroupLayout(
  cards: {/* 4 KPI cards */},
  cardWidth: "NARROW_PLUS"  /* 320px × 4 = 1280px, exceeds viewport */
)

/* ✅ CORRECT - 4 KPIs with NARROW fits on 1 row */
a!cardGroupLayout(
  cards: {/* 4 KPI cards */},
  cardWidth: "NARROW"  /* 240px × 4 = 960px, fits comfortably */
)

/* ❌ WRONG - 2 charts with MEDIUM leaves too much whitespace */
a!cardGroupLayout(
  cards: {/* 2 chart cards */},
  cardWidth: "MEDIUM"  /* 400px × 2 = 800px, charts too small */
)

/* ✅ CORRECT - 2 charts with WIDE uses space better */
a!cardGroupLayout(
  cards: {/* 2 chart cards */},
  cardWidth: "WIDE"  /* 800px × 2 = 1600px, but wraps responsively */
)
```

**Tip:** When in doubt, test with different viewport sizes or start with the recommended width and adjust based on visual appearance.

### Background Color Best Practices

**Modern Approach: Favor White Backgrounds with Decorative Accents**

Using different colored backgrounds for each card section creates a dated, visually cluttered appearance. Instead, prefer neutral card backgrounds with selective use of decorative elements for visual interest.

**❌ Avoid: Rainbow Dashboard**
```sail
/* DON'T: Different colored cards throughout the page */
a!cardGroupLayout(
  cards: {
    a!cardLayout(
      contents: {/* KPI content */},
      style: "INFO",      /* Blue card */
      shape: "ROUNDED"
    ),
    a!cardLayout(
      contents: {/* Chart content */},
      style: "SUCCESS",   /* Green card */
      shape: "ROUNDED"
    ),
    a!cardLayout(
      contents: {/* Grid content */},
      style: "#FFE5E5",   /* Pink card */
      shape: "ROUNDED"
    )
  }
)
```
**✅ Preferred: Plain White Card with Simple Title**
```sail
  a!cardLayout(
    contents: {
      a!headingField(
        text: "Section Title",
        size: "MEDIUM_PLUS",
        headingTag: "H2",
        fontWeight: "SEMI_BOLD",
        marginBelow: "STANDARD"
      ),
      /* Rest of card contents here */
    },
    height: "AUTO",
    style: "NONE",
    shape: "ROUNDED",
    padding: "STANDARD",
    marginBelow: "STANDARD"
  )
```

**✅ Optional: White Card with Decorative Elements**
```sail
/* DO: To draw attention to certain cards, add decorativeBar and/or stamp */
a!cardLayout(
  contents: {
    a!sideBySideLayout(
      items: {
        a!sideBySideItem(
          item: a!stampField( /* colored stamp ahead of card title for pop of color */
            icon: "graduation-cap",
            backgroundColor: "#0D4FD6",
            size: "TINY"
          ),
          width: "MINIMIZE"
        ),
        a!sideBySideItem(
          item: a!headingField(
            text: "Student Information",
            size: "MEDIUM_PLUS",
            headingTag: "H1",
            fontWeight: "SEMI_BOLD",
            marginBelow: "NONE"
          )
        )
      },
      alignVertical: "MIDDLE",
      marginBelow: "MORE"
    ),
    /* Rest of card contents here */
  },
  style: "NONE",                    /* Clean white background */
  decorativeBarPosition: "TOP",   /* Colored accent bar on top */
  decorativeBarColor: "#0D4FD6",    /* Blue accent */
  shape: "ROUNDED",
  padding: "STANDARD"
)
```

**Key Guidelines:**
- ✅ Default to `style: "NONE"` (white) or `style: "STANDARD"` (light gray)
- ✅ AS NEEDED, add visual interest with `decorativeBarPosition` and `decorativeBarColor`
- ✅ AS NEEDED, use `stampField` components to add colorful accents
- ✅ Reserve colored backgrounds (INFO, SUCCESS, WARN, ERROR) for functional purposes like message banners
- ❌ Avoid using different colored backgrounds for each content section

## Avoiding Redundant Card Nesting

When a page section's content is already a collection of cards (like a cardGroupLayout or list of cardLayouts), **do not wrap them in an additional parent card**. This creates excessive boxiness and visual clutter.

Instead, place the section title and card collection directly on the page background. Use the headerContentLayout's `backgroundColor` parameter to create subtle contrast.

### ❌ WRONG: Wrapping Card Collections in Parent Cards
```sail
a!headerContentLayout(
  contents: {
    /* DON'T: Card wrapping a cardGroupLayout */
    a!cardLayout(
      contents: {
        a!headingField( /* Section title */
          text: "Key Metrics",
          size: "MEDIUM_PLUS",
          headingTag: "H2",
          fontWeight: "SEMI_BOLD",
          marginBelow: "STANDARD"
        ),
        a!cardGroupLayout( /* Set of KPI cards */
          cards: {
            a!cardLayout(contents: {/* KPI 1 */}, style: "NONE", shape: "ROUNDED"),
            a!cardLayout(contents: {/* KPI 2 */}, style: "NONE", shape: "ROUNDED"),
            a!cardLayout(contents: {/* KPI 3 */}, style: "NONE", shape: "ROUNDED")
          },
          cardWidth: "NARROW_PLUS"
        )
      },
      style: "NONE",
      shape: "ROUNDED",
      padding: "STANDARD",
      marginBelow: "STANDARD"
    )
  },
  backgroundColor: "#F5F6F8"
)
```
**Problem:** Creates a "box within boxes" effect - the outer card adds unnecessary visual weight when the inner cards already provide structure.

### ✅ RIGHT: Section Title + Card Collection on Page Background
```sail
a!headerContentLayout(
  contents: {
    /* DO: Section title and cards directly on page background */
    a!headingField( /* Section title */
      text: "Key Metrics",
      size: "MEDIUM_PLUS",
      headingTag: "H2",
      fontWeight: "SEMI_BOLD",
      marginBelow: "STANDARD"
    ),
    a!cardGroupLayout(
      cards: {
        a!cardLayout(contents: {/* KPI 1 */}, style: "NONE", shape: "ROUNDED", showBorder: true),
        a!cardLayout(contents: {/* KPI 2 */}, style: "NONE", shape: "ROUNDED", showBorder: true),
        a!cardLayout(contents: {/* KPI 3 */}, style: "NONE", shape: "ROUNDED", showBorder: true)
      },
      cardWidth: "NARROW_PLUS"
    )
  },
  backgroundColor: "#F5F6F8",  /* Page background creates contrast */
  contentsPadding: "MORE"
)
```
### Key Takeaway
**Cards containing cards = Remove the outer wrapper.** Let section titles and card collections breathe on the page background.

## Layout Patterns

### Dashboard Cards (Using cardGroupLayout)
```sail
a!cardGroupLayout(
  cards: {
    a!cardLayout(
      contents: { /* KPI content */ },
      style: "NONE",
      shape: "ROUNDED",
      padding: "STANDARD"
    ),
    a!cardLayout(
      contents: { /* Chart content */ },
      style: "NONE", 
      shape: "ROUNDED",
      padding: "STANDARD"
    )
  },
  cardWidth: "NARROW_PLUS",
  spacing: "STANDARD"
)
```

### Mixed Width Cards (Using columnsLayout)
```sail
a!columnsLayout(
  columns: {
    a!columnLayout(
      contents: a!cardLayout(
        contents: { /* Featured content */ },
        style: "ACCENT",
        shape: "ROUNDED",
        padding: "MORE"
      ),
      width: "WIDE"
    ),
    a!columnLayout(
      contents: a!cardLayout(
        contents: { /* Secondary content */ },
        style: "STANDARD",
        shape: "ROUNDED", 
        padding: "STANDARD"
      ),
      width: "MEDIUM"
    )
  }
)
```

### Status Cards
```sail
a!cardLayout(
  contents: { /* Status information */ },
  style: "SUCCESS",  /* or WARN, ERROR based on status */
  decorativeBarPosition: "TOP",
  decorativeBarColor: "POSITIVE",
  padding: "STANDARD",
  shape: "ROUNDED"
)
```

### Clickable Cards
```sail
a!cardLayout(
  contents: { /* Card content */ },
  link: a!dynamicLink(
    value: /* some value */,
    saveInto: /* target variable */
  ),
  showShadow: true,  /* Indicates interactivity */
  style: "STANDARD",
  shape: "ROUNDED"
)
```

## Tailwind CSS to SAIL Card Styling Examples

### Basic White Card with Shadow

**Tailwind CSS:**
```html
<div class="bg-white rounded-lg shadow-md p-6">
  <!-- content -->
</div>
```

**SAIL Equivalent:**
```sail
a!cardLayout(
  contents: { /* content */ },
  style: "NONE",          /* White background */
  shape: "ROUNDED",       /* Rounded corners */
  showShadow: true,       /* Drop shadow */
  padding: "MORE"         /* Generous padding */
)
```

### Card with Colored Background

**Tailwind CSS:**
```html
<div class="bg-blue-50 border border-blue-200 rounded-lg p-4">
  <!-- content -->
</div>
```

**SAIL Equivalent:**
```sail
a!cardLayout(
  contents: { /* content */ },
  style: "#EFF6FF",       /* Light blue background */
  shape: "ROUNDED",
  showBorder: true,
  borderColor: "#BFDBFE", /* Blue border */
  padding: "STANDARD"
)
```

### Borderless Card

**Tailwind CSS:**
```html
<div class="bg-gray-100 rounded-xl p-6 border-0">
  <!-- content -->
</div>
```

**SAIL Equivalent:**
```sail
a!cardLayout(
  contents: { /* content */ },
  style: "#F3F4F6",       /* Light gray background */
  shape: "ROUNDED",       /* Extra rounded corners */
  showBorder: false,      /* No border */
  padding: "MORE"
)
```
### Dark Card

**Tailwind CSS:**
```html
<div class="bg-gray-800 text-white rounded-lg p-6 shadow-lg">
  <!-- content -->
</div>
```

**SAIL Equivalent:**
```sail
a!cardLayout(
  contents: { /* content */ },
  style: "#1F2937",        /* Dark gray background */
  shape: "ROUNDED",
  showShadow: true,
  padding: "MORE"
)
```

### Minimal Square Card

**Tailwind CSS:**
```html
<div class="bg-white border border-gray-200 p-4">
  <!-- content -->
</div>
```

**SAIL Equivalent:**
```sail
a!cardLayout(
  contents: { /* content */ },
  style: "NONE",           /* White background */
  shape: "SQUARED",        /* No rounded corners */
  showBorder: true,
  borderColor: "#E5E7EB",  /* Gray border */
  padding: "STANDARD"
)
```

## Validation Checklist
- [ ] Consistent padding and spacing across similar cards
- [ ] Chosen appropriate layout method (cardGroupLayout vs columnsLayout)

## Common Mistakes to Avoid
❌ **Wrong:**
- Inconsistent padding/spacing across related cards
- Using different colored backgrounds for each card section (creates dated appearance)
- Using bright colors (SUCCESS, ERROR, INFO) without semantic meaning
- Using cardGroupLayout when cards need different widths

✅ **Correct:**
- Consistent styling across similar card types
- Neutral backgrounds with decorative bars and stamps for visual interest
- Meaningful use of color styles and decorative elements
- Choosing the right layout method for the specific use case