# SAIL Card Choice Field Usage Instructions

## Overview
CardChoiceField displays selectable options as styled cards rather than traditional radio buttons or checkboxes. Each card can display rich content including text, icons, and colored elements using predefined templates. It's ideal for visual selection interfaces where options need more context than simple text labels.

## ⚠️ CRITICAL DECISION POINT: CardChoiceField vs Custom Pattern

### Use CardChoiceField When:
- Cards only need **text + optional icon**
- No conditional elements (nothing shows/hides based on data)
- No custom styling beyond what templates provide
- Standard selection highlighting is sufficient
- Cards have identical layouts across all options

### Use Custom Card Choice Pattern When:
- Cards need **multiple component types** (stamps + tags + rich text + icons)
- Cards need **conditional elements** (some items show tags, others don't)
- Cards need **custom selection indicators** (checkmarks, custom highlighting)
- Cards need **complex layouts** with 3+ distinct content areas
- Cards need **custom background colors** or **borders**
- Cards have **different layouts** for different options

## Quick Identification Guide

**Look for these signals that require Custom Pattern:**
- ✋ **Multiple component types in one card** (e.g., stamp + text + tag + icon)
- ✋ **"Show when" conditions** (e.g., "only show 'NO FEES' tag for PayPal option")
- ✋ **Custom selection feedback** (e.g., custom checkmarks, colored borders)
- ✋ **Rich layouts** (e.g., sideBySideLayout with 3+ items)

**CardChoiceField is sufficient when:**
- ✅ **Simple content** (title + description + optional icon)
- ✅ **Identical structure** across all cards
- ✅ **No conditional logic** in card content
- ✅ **Standard selection behavior** is acceptable

## When to Use Each Alternative

### ✅ Use CardChoiceField For:
- **Visual option selection** - When choices need images, icons, or rich formatting
- **Product/service selection** - Choosing between plans, features, or configurations  
- **Category selection** - When options have descriptions or visual identifiers
- **Dashboard widgets** - Letting users pick chart types, layouts, or views
- **Multi-attribute choices** - Options with titles, descriptions, and metadata

### ✅ Use Custom Card Choice Pattern For:
- **Complex card layouts** - When built-in templates are too restrictive
- **Mixed content types** - Cards with stamps, tags, and multiple text elements
- **Conditional elements** - Showing/hiding parts of cards based on data
- **Custom selection indicators** - Beyond the standard selection highlighting
- **Rich interactive cards** - Cards with multiple clickable areas or complex layouts

### ❌ Use Simple Form Fields Instead:
- **Simple text choices** → Use `a!radioButtonField()` or `a!checkboxField()`
- **Large lists (15+ items)** → Use `a!dropdownField()` with search
- **Hierarchical choices** → Use `a!dropdownField()` with grouped options
- **Single-word selections** → Use `a!tagField()` for compact display

## CardChoiceField Implementation

### Core Parameters
- **data**: The list of options using a!map() for inline data
- **value**: Currently selected card values (array for multiple selections)
- **maxSelections**: Maximum number of cards that can be selected (1 for radio behavior, >1 for checkbox behavior)
- **saveInto**: Variable updates when selection changes
- **cardTemplate**: Defines the visual layout and content of each card
- **showShadow**: Whether cards display drop shadows (default: false)
- **spacing**: Space between cards (STANDARD recommended, MORE, EVEN_MORE)
- **align**: Card alignment (START recommended, CENTER, END)
- **disabled**: Prevents user interaction (grayed out state)

### Available Templates (Valid Parameters Only)

#### a!cardTemplateBarTextStacked()
```sail
a!cardTemplateBarTextStacked(
  id: fv!data.id,                    /* Required: Unique identifier */
  primaryText: fv!data.title,        /* Main heading */
  secondaryText: fv!data.description, /* Subtitle */
  icon: fv!data.icon                 /* Optional icon */
)
```

#### a!cardTemplateBarTextJustified()
```sail
a!cardTemplateBarTextJustified(
  id: fv!data.id,                    /* Required: Unique identifier */
  primaryText: fv!data.title,        /* Left-aligned main text */
  secondaryText: fv!data.description, /* Left-aligned subtitle */
  icon: fv!data.icon                 /* Optional icon */
)
```

#### a!cardTemplateTile()
```sail
a!cardTemplateTile(
  id: fv!data.id,                    /* Required: Unique identifier */
  primaryText: fv!data.title,        /* Main text */
  secondaryText: fv!data.subtitle,   /* Optional subtitle */
  icon: fv!data.icon                 /* Optional icon */
)
```

### CardChoiceField Examples

#### Standard Plan Selection
```sail
a!cardChoiceField(
  label: "Choose Your Plan",
  data: {
    a!map(id: "basic", title: "Basic", description: "Perfect for individuals"),
    a!map(id: "pro", title: "Professional", description: "Great for small teams"),
    a!map(id: "enterprise", title: "Enterprise", description: "For large organizations")
  },
  cardTemplate: a!cardTemplateBarTextStacked(
    id: fv!data.id,
    primaryText: fv!data.title,
    secondaryText: fv!data.description
  ),
  value: local!selectedPlan,
  saveInto: local!selectedPlan,
  maxSelections: 1,
  align: "START",
  spacing: "STANDARD"
)
```

#### Simple Category Selection
```sail
a!cardChoiceField(
  label: "Content Type",
  data: {
    a!map(id: "document", title: "Document", icon: "file-text-o"),
    a!map(id: "image", title: "Image", icon: "picture-o"),
    a!map(id: "video", title: "Video", icon: "video-camera"),
    a!map(id: "audio", title: "Audio", icon: "music")
  },
  cardTemplate: a!cardTemplateTile(
    id: fv!data.id,
    primaryText: fv!data.title,
    icon: fv!data.icon
  ),
  value: local!contentType,
  saveInto: local!contentType,
  maxSelections: 1,
  align: "START",
  spacing: "STANDARD"
)
```

## Custom Card Choice Pattern Implementation

### When CardChoiceField Templates Are Insufficient
Use this approach when you need:
- Multiple component types in each card
- Conditional elements based on data
- Custom selection styling
- Complex layouts

### Basic Custom Pattern Structure
```sail
a!localVariables(
  local!options: {
    /* Your option data */
  },
  local!selectedId: "default_value",
  
  a!cardGroupLayout(
    cards: a!forEach(
      items: local!options,
      expression: a!cardLayout(
        contents: {
          /* Custom card content using sideBySideLayout */
        },
        link: a!dynamicLink(
          value: fv!item.id,
          saveInto: local!selectedId
        ),
        style: if(
          local!selectedId = fv!item.id,
          "#EFF6FF",  /* Selected state background */
          "#FFFFFF"   /* Unselected state background */
        ),
        borderColor: if(
          local!selectedId = fv!item.id,
          "ACCENT",   /* Selected state border */
          "STANDARD"  /* Unselected state border */
        )
      )
    )
  )
)
```

### Complex Custom Pattern Example
```sail
a!localVariables(
  local!paymentMethods: {
    a!map(
      id: "card", 
      name: "Credit Card", 
      description: "Visa, Mastercard, American Express", 
      icon: "credit-card", 
      color: "#3B82F6"
    ),
    a!map(
      id: "paypal", 
      name: "PayPal", 
      description: "Pay with your PayPal account", 
      icon: "shield", 
      color: "#EAB308"
    ),
    a!map(
      id: "bank", 
      name: "Bank Transfer", 
      description: "Direct bank transfer", 
      icon: "building", 
      color: "#059669"
    )
  },
  local!selectedPaymentId: "card",

  a!cardGroupLayout(
    cards: a!forEach(
      items: local!paymentMethods,
      expression: a!cardLayout(
        contents: {
          a!sideBySideLayout(
            items: {
              a!sideBySideItem(
                item: a!stampField(
                  icon: fv!item.icon,
                  backgroundColor: fv!item.color,
                  contentColor: "#FFFFFF",
                  size: "TINY",
                  shape: "ROUNDED",
                  labelPosition: "COLLAPSED"
                ),
                width: "MINIMIZE"
              ),
              a!sideBySideItem(
                item: a!richTextDisplayField(
                  value: {
                    a!richTextItem(
                      text: fv!item.name,
                      style: "STRONG",
                      size: "STANDARD"
                    ),
                    char(10),
                    a!richTextItem(
                      text: fv!item.description,
                      color: "SECONDARY",
                      size: "SMALL"
                    )
                  },
                  labelPosition: "COLLAPSED"
                ),
                width: "AUTO"
              ),
              a!sideBySideItem(
                item: a!tagField(
                  tags: a!tagItem(
                    text: "NO FEES",
                    backgroundColor: "POSITIVE"
                  ),
                  size: "SMALL",
                  labelPosition: "COLLAPSED"
                ),
                width: "MINIMIZE",
                showWhen: fv!item.id = "paypal"
              ),
              a!sideBySideItem(
                item: a!richTextDisplayField(
                  value: a!richTextIcon(
                    icon: "check",
                    color: "ACCENT",
                    size: "MEDIUM"
                  ),
                  labelPosition: "COLLAPSED"
                ),
                width: "MINIMIZE",
                showWhen: local!selectedPaymentId = fv!item.id
              )
            },
            alignVertical: "MIDDLE",
            spacing: "STANDARD"
          )
        },
        link: a!dynamicLink(
          value: fv!item.id,
          saveInto: local!selectedPaymentId
        ),
        style: if(
          local!selectedPaymentId = fv!item.id,
          "#EFF6FF",
          "#FFFFFF"
        ),
        borderColor: if(
          local!selectedPaymentId = fv!item.id,
          "ACCENT",
          "STANDARD"
        ),
        shape: "ROUNDED",
        padding: "STANDARD",
        showShadow: true
      )
    ),
    cardWidth: "MEDIUM",
    spacing: "STANDARD"
  )
)
```

## Decision Checklist

**Before choosing CardChoiceField, verify:**
- [ ] Each card only needs text + optional icon
- [ ] All cards have identical structure
- [ ] No conditional elements needed
- [ ] Standard selection highlighting is sufficient
- [ ] Built-in templates can handle the layout

**If ANY of the above is false, use Custom Card Choice Pattern instead.**

**Template Selection for CardChoiceField:**
- **Simple layout** → Tile template
- **Stacked text** → BarTextStacked template  
- **Justified text** → BarTextJustified template

Remember: CardChoiceField is simpler and should be preferred when it meets requirements. Only use Custom Pattern when CardChoiceField templates cannot achieve the desired design.