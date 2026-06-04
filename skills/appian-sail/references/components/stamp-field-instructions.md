# SAIL StampField Usage Instructions

## ⚠️ CRITICAL: Icon Parameter Validation

**BEFORE using ANY icons in stampField, you MUST:**
- [ ] Read `/ui-guidelines/reference/rich-text-icon-aliases.md` in full to see all valid icon names
- [ ] ONLY use icon names that appear exactly in that file
- [ ] NEVER guess icon names - always verify first

**The `icon` parameter accepts ONLY valid icon aliases.** Using invalid icon names will cause validation errors. There are ~1000 valid icons - guessing will fail.

---

## Overview
StampField creates an icon and/or text displayed on a colored background shape. It's designed for creating badges, status indicators, avatars, and decorative icons that need visual emphasis with a background.

## When to Use StampField vs Alternatives

### ❌ Don't Use StampField For:
- **Plain icons without backgrounds** → Use `a!richTextIcon()` instead
- **Small badges (less than 40px wide)** → Use `a!richTextIcon()` instead
- **Icons inline with text** → Use `a!richTextIcon()` in `a!richTextDisplayField()`
- **Tags** → Use `a!tagItem)` in `a!tagField()`
- **Large content areas** → Use `a!cardLayout()` instead
- **Complex multi-element badges** → Use `a!cardLayout()` with custom styling

## Core StampField Parameters

### Icon Parameter
- **icon**: Valid icon alias string from `/ui-guidelines/reference/rich-text-icon-aliases.md`
- **⚠️ MANDATORY**: Look up icon name in aliases file BEFORE use
- **❌ DO NOT GUESS** icon names - they must match exactly

### Shape Options
- **ROUNDED**: Perfect circle (best for avatars, status dots)
- **SEMI_ROUNDED**: Rounded corners (modern badge appearance)  
- **SQUARED**: Sharp corners (traditional badge style)

### Size Options with Dimensions
- **TINY**: 40px (often the most appropriate size)
- **SMALL**: 60px
- **MEDIUM**: 85px 
- **LARGE**: 140px (hero elements, large profile pictures)

### Background Color Options
- **ACCENT**: Default theme color (usually blue)
- **POSITIVE**: Green (success, active, approved)
- **NEGATIVE**: Red (error, rejected, critical)
- **SECONDARY**: Gray (inactive, neutral, secondary info)
- **TRANSPARENT**: No background (rare use case)
- **Hex values**: Custom colors like "#3B82F6"

### Content Color Options
- **STANDARD**: Default text color (usually dark on light backgrounds)
- **ACCENT**: Theme accent color
- **POSITIVE**: Green text
- **NEGATIVE**: Red text
- **Hex values**: Custom text colors

## Common Patterns

### 1. Status Badge
```sail
a!stampField(
  icon: "check",
  backgroundColor: "POSITIVE",
  contentColor: "STANDARD",
  shape: "ROUNDED",
  size: "TINY"
)
```

### 2. User Avatar with Initials
```sail
a!stampField(
  text: "CT",
  backgroundColor: "#3B82F6",
  contentColor: "#FFFFFF", 
  shape: "ROUNDED",
  size: "TINY"
)
```

### 3. Category Badge
```sail
a!stampField(
  text: "VIP",
  backgroundColor: "#7C3AED",
  contentColor: "#FFFFFF",
  shape: "SEMI_ROUNDED", 
  size: "TINY"
)
```

## Tailwind CSS to SAIL StampField Examples

### Circular Status Badge

**Tailwind CSS:**
```html
<span class="inline-flex items-center justify-center w-10 h-10 bg-green-500 text-white rounded-full text-xs">
  ✓
</span>
```

**SAIL Equivalent:**
```sail
a!stampField(
  icon: "check",
  backgroundColor: "#00c951",
  contentColor: "STANDARD", 
  shape: "ROUNDED",
  size: "TINY"
)
```

### User Avatar Circle

**Tailwind CSS:**
```html
<div class="w-10 h-10 bg-blue-500 rounded-full flex items-center justify-center text-white font-medium text-sm">
  JD
</div>
```

**SAIL Equivalent:**
```sail
a!stampField(
  text: "JD",
  backgroundColor: "#3B82F6",
  contentColor: "#FFFFFF",
  shape: "ROUNDED", 
  size: "TINY"
)
```

### Square Icon Badge

**Tailwind CSS:**
```html
<div class="w-8 h-8 bg-gray-200 rounded-md flex items-center justify-center">
  <svg class="w-4 h-4 text-gray-600">...</svg>
</div>
```

**SAIL Equivalent:**
```sail
a!stampField(
  icon: "cog",
  backgroundColor: "#E5E7EB",
  contentColor: "#4B5563",
  shape: "SEMI_ROUNDED",
  size: "TINY"
)
```

## Best Practices

### ✅ DO:
- **ALWAYS read `/ui-guidelines/reference/rich-text-icon-aliases.md` before using ANY icon parameter**
- Choose shape based on content: ROUNDED for avatars/status, SEMI_ROUNDED for modern badges
- Use TINY size for most UI elements to avoid overwhelming the interface
- Provide tooltips for icon-only stamps
- Use consistent sizing within the same UI section

### ❌ DON'T:
- **Guess icon names - ALWAYS verify in aliases file first**
- Use stamps for plain icons without backgrounds (use richTextIcon instead)
- Make stamps too large (MEDIUM/LARGE should be rare)
- Put complex content in stamps (use cardLayout instead)
- Mix different stamp sizes randomly in the same area

## Common Conversion Mistakes

### ❌ WRONG - Using Stamp for Plain Icon:
```html
<!-- Tailwind: Just an icon, no background -->
<svg class="w-5 h-5 text-gray-500">...</svg>
```
```sail
/* Don't use stampField here */
a!stampField(icon: "cog", backgroundColor: "TRANSPARENT")

/* Use richTextIcon instead */
a!richTextDisplayField(
  value: a!richTextIcon(icon: "cog", color: "#6B7280")
)
```

### ✅ CORRECT - Choosing Right Alternative:
```html
<!-- Complex badge with multiple elements -->
<div class="flex items-center gap-2 px-3 py-1 bg-blue-50 rounded-lg">
  <svg>...</svg>
  <span>Admin</span>
  <span class="font-bold">Level 5</span>
</div>
```

Use `a!cardLayout()` with `a!sideBySideLayout()` inside, not `a!stampField()`.

## Quick Decision Guide

**"Does the element need a colored background shape?"**
- Yes → **StampField**
- No → **RichTextIcon**

Remember: StampField is for simple badges and indicators with background shapes. When in doubt, start with TINY size and ROUNDED shape for maximum versatility.

---

## ⚠️ MANDATORY VALIDATION CHECKLIST

Before using stampField in your code:

### Icon Validation (CRITICAL):
- [ ] **IF using `icon` parameter:** Read `/ui-guidelines/reference/rich-text-icon-aliases.md` FIRST
- [ ] **Verify icon name exists** in aliases file using exact string match
- [ ] **DO NOT GUESS** icon names based on other frameworks (Font Awesome, Material Icons, etc.)
- [ ] Common mistake: "chart-bar" ❌ → Correct: "bar-chart" ✅ (always check the file!)

### Parameter Validation:
- [ ] `shape` is one of: `"ROUNDED"`, `"SEMI_ROUNDED"`, `"SQUARED"`
- [ ] `size` is one of: `"TINY"`, `"SMALL"`, `"MEDIUM"`, `"LARGE"`
- [ ] `backgroundColor` is semantic color or valid 6-character hex code
- [ ] `contentColor` is semantic color or valid 6-character hex code
- [ ] `labelPosition` is set to `"COLLAPSED"` when appropriate

### Design Validation:
- [ ] Using stampField for appropriate use case (needs background shape)
- [ ] Not using for plain icons (use richTextIcon instead)
- [ ] Size is appropriate (TINY for most UI elements)
- [ ] Colors have sufficient contrast for accessibility