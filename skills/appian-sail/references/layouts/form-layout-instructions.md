# SAIL FormLayout Usage Instructions

## Overview
FormLayout is the top-level container for form-based interfaces in SAIL. It provides built-in width constraints and form-specific styling without requiring additional layout components for basic header styling, setting page background color, or constraining content width.

✅ Use FormLayout for single-step forms
❌ Don't use FormLayout for multi-step wizards-use wizardLayout instead

```sail
/* GOOD - Data entry form */
a!formLayout(
  titleBar: "Employee Registration",
  contentsWidth: "MEDIUM",
  contents: {
    a!textField(label: "First Name", value: local!firstName, saveInto: local!firstName),
    a!textField(label: "Last Name", value: local!lastName, saveInto: local!lastName),
    a!dropdownField(label: "Department", ...)
  },
  buttons: a!buttonLayout(
    primaryButtons: a!buttonWidget(label: "Submit", submit: true())
  )
)
```
## Core FormLayout Parameters
- **titleBar**: Text or CardLayout for form header
- **contents**: Array of form components and layouts
- **contentsWidth**: Width of form contents (not including the header)
- **buttons**: ButtonLayout for form actions
- **backgroundColor**: Form background color
- **validations**: Form-level error messages
- **focusOnFirstInput**: Optional automatic focus management

## ContentsWidth - Limiting width of form contents

### Available ContentsWidth Values
Explicitly set the right contentsWidth for form contents:
- **FULL**: Edge-to-edge, uses full container width
- **WIDE**: ~800px maximum width, centered
- **MEDIUM**: ~400px maximum width, centered  
- **NARROW** (default): ~240px maximum width, centered - USE RARELY, TOO NARROW for all but the simplest single-column form contents
- **EXTRA_NARROW**: ~80px maximum width, centered - USE RARELY, TOO NARROW

### When to Use Each Width
```sail
a!formLayout(
  contentsWidth: "FULL",        // To maximize horizontal space or when using columnsLayout inside form contents to govern layout
  contentsWidth: "WIDE",        // Dashboard-style forms with multiple columns
  contentsWidth: "MEDIUM",      // Standard single-column forms
  contentsWidth: "NARROW",      // Simple login/signup forms
  contentsWidth: "EXTRA_NARROW" // Rarely used - too narrow
)
```

## ✅ CORRECT APPROACHES

### Approach 1: Use ContentsWidth for Simple Width Control
```sail
a!formLayout(
  contentsWidth: "MEDIUM",  // ✅ Simple, clean width constraint
  contents: {
    a!textField(...),
    a!paragraphField(...),
    a!sectionLayout(...)
  }
)
```

### Approach 2: Use FULL ContentsWidth + ColumnsLayout for Complex Layouts
```sail
a!formLayout(
  contentsWidth: "FULL",    // ✅ Let columnsLayout handle width
  contents: {
    a!columnsLayout(
      columns: {
        a!columnLayout(width: "AUTO"),           /* Left margin */
        a!columnLayout(width: "MEDIUM", contents: {...}), /* Main content */
        a!columnLayout(width: "NARROW", contents: {...}), /* Sidebar */
        a!columnLayout(width: "AUTO")            /* Right margin */
      }
    )
  }
)
```

## Decision Matrix: ContentsWidth vs ColumnsLayout

| Scenario | Solution | Reasoning |
|----------|----------|-----------|
| Single-column form | `contentsWidth: "MEDIUM"` | Simple, built-in constraint |
| Multi-column layout | `contentsWidth: "FULL"` + columnsLayout | Need column control |
| Full-width dashboard | `contentsWidth: "FULL"` | No constraints needed |
| Narrow login form | `contentsWidth: "NARROW"` | Very simple mini forms in cards |

## TitleBar Options - Different Ways to Add Headers

FormLayout provides multiple ways to create headers, from simple text to complex styled headers with images and sidebars.

### 1. Simple Text String
The most basic approach - just pass a text string to the titleBar parameter.

```sail
a!formLayout(
  titleBar: "Employee Registration",  /* Simple text string */
  contentsWidth: "MEDIUM",
  contents: {
    a!textField(label: "First Name", ...),
    a!textField(label: "Last Name", ...)
  }
)
```

### 2. Header Templates

Appian provides pre-built header templates for common design patterns.

#### a!headerTemplateSimple
A clean, simple header that aligns with content width.

```sail
a!formLayout(
  titleBar: a!headerTemplateSimple(
    title: "Project Settings",
    secondaryText: "Configure your project preferences and team access",
    titleColor: "#1F2937",
    secondaryTextColor: "#6B7280",
    stampIcon: "cog",
    stampColor: "ACCENT"
  ),
  contentsWidth: "MEDIUM",
  contents: {...}
)
```
**Parameters:**
- `title`: Main heading text
- `secondaryText`: Subtitle or description
- `titleColor`: Color for main title (hex or enumeration)
- `secondaryTextColor`: Color for subtitle
- `stampIcon`: Optional decorative icon
- `stampColor`: Color for the stamp icon

#### a!headerTemplateFull
A full-width header with background color that spans the entire screen.

```sail
a!formLayout(
  titleBar: a!headerTemplateFull(
    title: "Annual Performance Review",
    secondaryText: "Fiscal Year 2024 Employee Evaluation",
    backgroundColor: "#2563EB",
    titleColor: "#FFFFFF",
    secondaryTextColor: "#DBEAFE",
    stampIcon: "star",
    stampColor: "#FFFFFF"
  ),
  contentsWidth: "WIDE",
  contents: {...}
)
```

**Parameters:**
- `title`: Main heading text
- `secondaryText`: Subtitle or description
- `backgroundColor`: Full-width background color
- `titleColor`: Color for main title
- `secondaryTextColor`: Color for subtitle
- `stampIcon`: Optional decorative icon
- `stampColor`: Color for the stamp icon

#### a!sidebarTemplate
Displays a sidebar on desktop that becomes a header on mobile/narrow screens.

```sail
a!formLayout(
  titleBar: a!sidebarTemplate(
    title: "Task Management",
    secondaryText: "Organize and track your daily tasks",
    image: a!webImage(source: "https://example.com/task-icon.png"),
    additionalContents: {
      a!richTextDisplayField(
        value: {
          a!richTextIcon(icon: "clock-o", color: "SECONDARY"),
          " ",
          a!richTextItem(text: "Due: Today", color: "SECONDARY", size: "SMALL")
        },
        labelPosition: "COLLAPSED"
      )
    }
  ),
  contentsWidth: "MEDIUM",
  contents: {...}
)
```

**Parameters:**
- `title`: Main heading text
- `secondaryText`: Subtitle or description
- `image`: Optional decorative image
- `additionalContents`: Extra components below the text

## Form Structure Best Practices

### Basic Form Structure
```sail
a!formLayout(
  titleBar: "Form Title",
  contentsWidth: "MEDIUM",
  contents: {
    a!cardLayout(
      contents: {
        a!sectionLayout(
          label: "Section 1",
          contents: {
            a!textField(...), 
            a!dropdownField(...) 
          }
        )
      },
      style: "NONE",
      showBorder: true,
      showShadow: false,
      shape: "ROUNDED",
      padding: "STANDARD"
    )
  },
  buttons: a!buttonLayout(
    primaryButtons: {
      a!buttonWidget(label: "Submit", submit: true())
    }
  )
)
```

### Complex, Multi-Column Form Structure
```sail
a!localVariables(
  /* Local variables for form inputs */
  local!firstName,
  local!lastName,
  local!email,
  local!phone,
  local!username,
  local!password,
  local!confirmPassword,
  a!formLayout(
    titleBar: a!headerTemplateFull(
      title: "Create Your Account",
      secondaryText: "Join our platform and start your journey today",
      backgroundColor: "#1C2C44",
      titleColor: "#FFFFFF",
      secondaryTextColor: "#E8F4FD"
    ),
    contentsWidth: "FULL", /* set to FULL because contents layout will be delegated to columnsLayout within contents */
    backgroundColor: "#F5F6F8",
    contents: {
      a!columnsLayout(
        columns: {
          /* Empty margin column */
          a!columnLayout(width: "AUTO"),
          /* Main content column */
          a!columnLayout(
            width: "WIDE",
            contents: {
              /* About You Card */
              a!cardLayout(
                contents: {
                  a!sectionLayout(
                    label: "About You",
                    labelColor: "STANDARD",
                    contents: {
                      a!columnsLayout(
                        columns: {
                          a!columnLayout(
                            contents: {
                              a!textField(
                                label: "First Name",
                                value: local!firstName,
                                saveInto: local!firstName,
                                required: true()
                              )
                            }
                          ),
                          a!columnLayout(
                            contents: {
                              a!textField(
                                label: "Last Name",
                                value: local!lastName,
                                saveInto: local!lastName,
                                required: true()
                              )
                            }
                          )
                        }
                      ),
                      a!textField(
                        label: "Username",
                        value: local!username,
                        saveInto: local!username,
                        required: true(),
                        placeholder: "Choose a unique username"
                      ),
                      a!columnsLayout(
                        columns: {
                          a!columnLayout(
                            contents: {
                              a!encryptedTextField(
                                label: "Password",
                                value: local!password,
                                saveInto: local!password,
                                required: true(),
                                placeholder: "Enter a secure password"
                              )
                            }
                          ),
                          a!columnLayout(
                            contents: {
                              a!encryptedTextField(
                                label: "Confirm Password",
                                value: local!confirmPassword,
                                saveInto: local!confirmPassword,
                                required: true(),
                                placeholder: "Re-enter your password"
                              )
                            }
                          )
                        }
                      )
                    }
                  )
                },
                style: "#FFFFFF",
                shape: "ROUNDED",
                showBorder: true,
                padding: "STANDARD",
                marginBelow: "STANDARD"
              ),
              /* Contact Information Card */
              a!cardLayout(
                contents: {
                  a!sectionLayout(
                    label: "Contact Information",
                    labelColor: "STANDARD",
                    contents: {
                      a!textField(
                        label: "Email Address",
                        value: local!email,
                        saveInto: local!email,
                        required: true(),
                        inputPurpose: "EMAIL",
                        placeholder: "your.email@example.com"
                      ),
                      a!textField(
                        label: "Phone Number",
                        value: local!phone,
                        saveInto: local!phone,
                        inputPurpose: "PHONE_NUMBER",
                        placeholder: "(555) 123-4567"
                      )
                    }
                  )
                },
                style: "#FFFFFF",
                shape: "ROUNDED",
                showBorder: true,
                padding: "STANDARD"
              )
            }
          ),
          /* Spacer column */
          a!columnLayout(width: "EXTRA_NARROW"),
          /* Additional info column with instructions */
          a!columnLayout(
            width: "MEDIUM",
            contents: {
              a!cardLayout(
                contents: {
                  a!sectionLayout(
                    label: "Getting Started",
                    labelColor: "STANDARD",
                    contents: {
                      a!richTextDisplayField(...)
                    }
                  )
                },
                style: "NONE",
                shape: "ROUNDED",
                showBorder: true,
                padding: "STANDARD"
              )
            }
          ),
          /* Empty margin column */
          a!columnLayout(width: "AUTO")
        }
      )
    },
    buttons: a!buttonLayout(
      primaryButtons: {
        a!buttonWidget(
          label: "Create Account",
          submit: true(),
          style: "SOLID",
          loadingIndicator: true()
        )
      },
      secondaryButtons: {
        a!buttonWidget(label: "Cancel", style: "LINK")
      }
    )
  )
)
```
## ‼️ MANDATORY Review of ContentsWidth Decision

`contentsWidth` must be explicitly set for `formLayout`

Do form contents need a complex, multi-column layout?
├─ YES → contentsWidth: "FULL" + columnsLayout inside contents
└─ NO → Use simple contentsWidth constraint:
   ├─ Standard single-column form → contentsWidth: "MEDIUM"
   └─ Wide form, such as one split into two AUTO columns → contentsWidth: "WIDE" 

❌ NEVER combine `contentsWidth: "MEDIUM"` (which is the default if not specified) with a layout `columnsLayout` inside form contents!