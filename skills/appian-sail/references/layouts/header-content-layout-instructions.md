# SAIL HeaderContentLayout Usage Instructions

## Overview
HeaderContentLayout creates full-width (edge-to-edge) page layouts with an optional header section and main content area. It's one of the three top-level layout options in SAIL and is ideal for modern, full-bleed designs.

## When to Use HeaderContentLayout

### Use HeaderContentLayout for:
- Pages that need full-width headers (hero sections, banners)
- Modern designs with edge-to-edge content
- Pages with prominent visual headers (images, videos, graphics)
- Landing pages or marketing-style interfaces
- Any page where you want content to extend to screen edges
- Can be used for pages without headers (just leave header parameter empty)

### Use FormLayout instead for:
- Simple forms

### Use PaneLayout instead for:
- Multi-pane interfaces with independent scrolling
- Full-height layouts where panes fill the entire screen
- Applications requiring sidebar navigation

## Structure and Parameters

### Basic Structure
```sail
a!headerContentLayout(
  header: /* Billboard or card layouts or completely omit header */,
  contents: {
    /* Main page content - any layouts/components */
  }
)
```

### Key Parameters
- `header`: Billboard, card, or list of billboards/cards (can be empty)
- `contents`: Array of components and layouts for main content
- `backgroundColor`: Page background color
- `contentsPadding`: Space around the contents area
- `isHeaderFixed`: Whether header stays at top when scrolling

## backgroundColor Options
- `"WHITE"` (default): Standard white background
- `"TRANSPARENT"`: Light gray background
- Any valid hex color: `"#F5F5F5"`
- Recommended: `"#F5F6F8"`

## Contents Padding Options
- `"NONE"`: No padding around contents
- `"EVEN_LESS"`: Minimal padding
- `"LESS"`: Reduced padding
- `"STANDARD"` (default): Normal padding
- `"MORE"`: Increased padding
- `"EVEN_MORE"`: Maximum padding

## ⚠️ CRITICAL RULES: Header Parameter

### Valid Header Components
The `header` parameter **ONLY accepts these components:**
- `a!cardLayout()` - Most common for standard headers
- `a!billboardLayout()` - For headers with background images/videos
- Empty `{}` or omit parameter - No header

### ❌ INVALID Header Components
**NEVER use these directly in header:**
- `a!sideBySideLayout()` - Must be wrapped in a!cardLayout()
- `a!richTextDisplayField()` - Must be wrapped in a!cardLayout()
- `a!buttonArrayLayout()` - Must be wrapped in a!cardLayout()
- Any other component - Must be wrapped in a!cardLayout()

```sail
/* ❌ WRONG - sideBySideLayout directly in header */
header: {
  a!sideBySideLayout(...)
}

/* ✅ RIGHT - Wrapped in cardLayout */
header: {
  a!cardLayout(
    contents: {
      a!sideBySideLayout(...)
    },
    style: "NONE",
    height: "AUTO",
    marginBelow: "STANDARD"
  )
}
```

### When to Use Each Option

**Header is not needed for design:**
- Just exclude the `header` parameter or use `header: {}`

**Header must show an image or video as its background:**
- Use `a!billboardLayout()` in `header`

**Default (standard page header with title, buttons, etc.):**
- Use `a!cardLayout()` in `header` containing your header components

## Standard Template

Use this as a baseline for a standard page with title bar header:

```sail
a!headerContentLayout(
  header: {
    a!cardLayout(
      contents: {
        a!headingField( /* Page Title */
          text: "Page Title",
          size: "LARGE",
          headingTag: "H1",
          fontWeight: "BOLD",
          marginBelow: "NONE
        )
      },
      height: "AUTO",
      style: "#2C3E50", /* Sophisticated slate header background */
      padding: "MORE", /* Simple headers look better with more padding */
      marginBelow: "NONE",
      showBorder: false
    )
  },
  contents: {
    a!cardLayout( /* Card wrapping main page contents */
      contents: {},
      style: "NONE",
      shape: "ROUNDED",
      padding: "STANDARD",
      showBorder: true,
      showShadow: false
    )
  },
  contentsPadding: "MORE",
  backgroundColor: "#F5F6F8" /* Set a page background color for greater contrast against white cards */
)
```


## Common Patterns

### 1. Hero Section with Colored Background (No Image)
```sail
a!headerContentLayout(
  header: a!cardLayout(
    contents: {
      a!richTextDisplayField(
        value: {
          a!richTextItem(
            text: "Get Started Today",
            size: "LARGE",
            style: "STRONG",
            color: "#FFFFFF"
          ),
          char(10),
          a!richTextItem(
            text: "Join thousands of users who trust our platform",
            size: "MEDIUM",
            color: "#FFFFFF"
          )
        },
        align: "LEFT",
        labelPosition: "COLLAPSED"
      )
    },
    style: "#2C3E50",
    showBorder: false,
    showShadow: false,
    height: "AUTO",
    padding: "MORE"
  ),
  contents: {
    /* Main content */
  }
)
```

### 2. No Header (Content Only - use to set custom page background color)
```sail
a!headerContentLayout(
  contents: {
    a!columnsLayout(
      columns: {
        a!columnLayout(width: "AUTO"),
        a!columnLayout(
          width: "EXTRA_WIDE",
          contents: {
            /* Centered content */
          }
        ),
        a!columnLayout(width: "AUTO")
      }
    )
  },
  backgroundColor: "#F8F9FA",
  contentsPadding: "MORE"
)
```

### 3. Lightweight header

If a boxy card header isn't needed, exclude the `header` parameter and put header contents (like the page title) directly in page `contents`:

```sail
a!headerContentLayout(
  contents: {
    a!headingField( /* Page title included in page contents, not header */
      text: "Page Title",
      size: "LARGE",
      headingTag: "H1",
      fontWeight: "BOLD",
      marginBelow: "MORE" /* Add space between title and page contents */
    ),
    a!cardLayout( /* Card wrapping main page contents */
      contents: {
        /* Page contents */
      },
      style: "NONE",
      shape: "ROUNDED",
      padding: "STANDARD",
      showBorder: true,
      showShadow: false
    )
  },
  contentsPadding: "MORE",
  backgroundColor: "#F5F6F8" /* Set a page background color for greater contrast against white cards */
)
```

## Billboard Layout with Overlay - CRITICAL RULES

### ⚠️ Billboard Overlay Requirement
The `overlay` parameter in `a!billboardLayout()` **MUST** use one of these wrapper functions:
- `a!fullOverlay()` - Covers entire billboard
- `a!barOverlay()` - Horizontal bar at top, middle, or bottom
- `a!columnOverlay()` - Vertical column at left, center, or right

**Components cannot be placed directly in the overlay parameter!**

### ❌ WRONG: Direct Component in Overlay
```sail
/* THIS WILL CAUSE AN ERROR */
a!billboardLayout(
  backgroundMedia: a!webImage(source: "hero.jpg"),
  overlay: {
    a!richTextDisplayField(...)  /* ERROR: Cannot use component directly */
  }
)

/* THIS WILL ALSO CAUSE AN ERROR */
a!billboardLayout(
  backgroundMedia: a!webImage(source: "hero.jpg"),
  overlay: a!richTextDisplayField(...)  /* ERROR: Must wrap in overlay type */
)
```

### ✅ CORRECT: Full Overlay with Centered Content
```sail
a!billboardLayout(
  backgroundMedia: a!webImage(
    source: "https://example.com/hero-image.jpg",
    altText: "Hero background"
  ),
  backgroundColor: "#1C2C44",
  height: "MEDIUM_PLUS",
  overlay: a!fullOverlay(  /* Must use overlay wrapper! */
    alignVertical: "MIDDLE",
    style: "SEMI_DARK",  /* Darkens background for text readability */
    contents: {
      a!richTextDisplayField(
        labelPosition: "COLLAPSED",
        value: {
          a!richTextItem(
            text: "Welcome to Our Platform",
            size: "LARGE",
            style: "STRONG",
            color: "#FFFFFF"
          )
        },
        align: "CENTER"
      )
    }
  )
)
```

### ✅ CORRECT: Bar Overlay at Bottom
```sail
a!billboardLayout(
  backgroundMedia: a!webImage(source: "banner.jpg"),
  height: "TALL",
  overlay: a!barOverlay(
    position: "BOTTOM",
    style: "DARK",
    contents: {
      a!richTextDisplayField(
        labelPosition: "COLLAPSED",
        value: {
          a!richTextItem(
            text: "New Product Launch",
            size: "MEDIUM",
            style: "STRONG",
            color: "#FFFFFF"
          )
        }
      )
    }
  )
)
```

### ✅ CORRECT: Column Overlay on Left
```sail
a!billboardLayout(
  backgroundMedia: a!webImage(source: "hero.jpg"),
  height: "TALL_PLUS",
  overlay: a!columnOverlay(
    position: "START",
    alignVertical: "MIDDLE",
    width: "MEDIUM",
    style: "SEMI_DARK",
    contents: {
      a!richTextDisplayField(
        labelPosition: "COLLAPSED",
        value: {
          a!richTextItem(
            text: "Transforming Business",
            size: "LARGE",
            style: "STRONG",
            color: "#FFFFFF"
          ),
          char(10),
          a!richTextItem(
            text: "Discover how we can help your organization succeed",
            color: "#FFFFFF"
          )
        }
      )
    }
  )
)
```

## Multiple Headers Example
```sail
a!headerContentLayout(
  header: {
    a!billboardLayout(
      backgroundMedia: a!webImage(source: "hero-bg.jpg"),
      overlay: a!fullOverlay(
        alignVertical: "MIDDLE",
        style: "SEMI_DARK",
        contents: {
          /* Hero section content with background image */
        }
      )
    ),
    a!cardLayout(
      contents: {
        /* Action bar - no background image */
      }
    )
  },
  contents: {
    /* Main content */
  }
)
```

## Best Practices

### ✅ DO:
- Use billboardLayout ONLY when you have a background image/video
- Use cardLayout for hero sections without background images
- Use cardLayout for toolbars, action bars, and simple headers
- (Optional) Combine with columnsLayout in contents for proper content margins
- Use appropriate contentsPadding for your design

### ❌ DON'T:
- Use billboardLayout without a background image (use cardLayout instead)
- Nest headerContentLayout inside other layouts
- Use when formLayout or paneLayout would be more appropriate
- Forget to add margins around content using columnsLayout (if necessary)

## MANDATORY VALIDATION CHECKLIST
- [ ] `header` is omitted OR contains ONLY `cardLayout` or `billboardLayout` at the top level
- [ ] `contentsPadding` has appropriate value for padding around page contents
- [ ] `cardLayout` in `header` has appropriate padding set
