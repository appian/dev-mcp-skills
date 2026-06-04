# SAIL ImageField Usage Instructions

## Overview
ImageField displays one or more images with flexible sizing and styling options.

## Core ImageField Parameters

### Size Options

The `size` parameter determines how images are displayed. Understanding these options is critical for proper image presentation.

#### Fixed Size Values
- **ICON**: 20x20 pixels (source images should be 40x40 pixels for retina displays)
- **ICON_PLUS**: 40x40 pixels
- **TINY**: 60x120 pixels
- **EXTRA_SMALL**: 80x160 pixels
- **SMALL**: 100x200 pixels
- **SMALL_PLUS**: 150x300 pixels
- **MEDIUM**: 200x400 pixels (default)
- **MEDIUM_PLUS**: 300x500 pixels
- **LARGE**: 400x600 pixels
- **LARGE_PLUS**: 600x800 pixels
- **EXTRA_LARGE**: 800x1000 pixels
- **GALLERY**: 240x80 pixels (wide format for horizontal galleries)

#### Special Size Value: FIT ⭐
**FIT is the recommended size for most full-width images.**

- **FIT**: Natural dimensions (images display at natural width or container width, whichever is smaller)

When `size` is set to `"FIT"`:
- Images display at either their **natural width** or the **width of the container**, whichever is smaller
- This allows images to fill the full width of their container
- Images will scale down to fit narrow containers but won't exceed their natural resolution
- **Best practice**: Use FIT when you want images to be responsive and fill available horizontal space

### Style Options

The `style` parameter controls how images are scaled and cropped:

#### STANDARD (Default)
- Images are **scaled down** to fit the size limit
- Preserves natural aspect ratio
- **Never scales images up** - displays at natural size if smaller than configured size
- Use for: Product images, document previews, general photos

#### AVATAR
- Images are **scaled down or up** to fit the size limit
- Preserves natural aspect ratio
- **Cropped in a circle** - perfect for profile pictures
- Use for: User profile images, team member photos, circular avatars, displaying initials

### Image Sources

Images can be provided using:
- `a!documentImage()` - For Appian document images
- `a!userImage()` - For user profile pictures (displays stylized initials when no photo is available)
- `a!webImage()` - For external web images

⚠️ **These are image sources, not standalone components.** Always wrap them in `a!imageField(images: ...)`.

## Common Patterns

### 1. Full-Width Responsive Image
```sail
a!imageField(
  images: a!webImage(
    source: "https://example.com/product.jpg"
  ),
  size: "FIT", /* Fills container width */
  labelPosition: "COLLAPSED"
)
```

### 2. User Avatar Gallery
```sail
a!imageField(
  label: "Team Members",
  size: "MEDIUM",
  style: "AVATAR", /* Circular cropping */
  images: a!forEach(
    items: {"john.doe", "jane.smith", "bob.jones"},
    expression: a!userImage(user: fv!item)
  )
)
```

### 3. Thumbnail Gallery with Click-to-View
```sail
a!imageField(
  label: "Product Photos",
  images: {
    a!webImage(source: "https://example.com/photo1.jpg"),
    a!webImage(source: "https://example.com/photo2.jpg"),
    a!webImage(source: "https://example.com/photo3.jpg")
  },
  size: "GALLERY",
  isThumbnail: true, /* Click to view larger */
  style: "STANDARD"
)
```

## Best Practices

### ✅ DO:
- **Use FIT size** when you want images to fill the full width of their container
- **Use AVATAR style** exclusively for circular profile pictures
- **Set isThumbnail: true** for galleries where users might want to click to see larger versions
- **Set labelPosition: "COLLAPSED"** when no label is needed

### ❌ DON'T:
- Use STANDARD style for circular avatars (use AVATAR instead)
- Use AVATAR style for non-square/circle images (will crop unexpectedly)
- Use FIT with low-resolution images (will appear pixelated when stretched)
- Forget to provide alt text via accessibilityText for screen readers