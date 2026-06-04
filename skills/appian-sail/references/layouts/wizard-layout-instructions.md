# SAIL WizardLayout Usage Instructions

## Overview
WizardLayout is the top-level container for multi-step forms in SAIL. It provides built-in step navigation, progress indicators, and automatic Next/Back button management to guide users through complex data collection processes.

✅ Use WizardLayout for multi-step forms with sequential data collection
❌ Don't use WizardLayout for single-step forms—use FormLayout instead

```sail
/* GOOD - Multi-step insurance quote request */
a!wizardLayout(
  titleBar: a!headerTemplateFull(
    title: "Request a Quote",
    secondaryText: "Customize your coverage",
    backgroundColor: "#434343"
  ),
  isTitleBarFixed: false,
  showTitleBarDivider: false,
  style: "DOT_VERTICAL",
  steps: {
    a!wizardStep(
      label: "About You",
      instructions: "Tell us about yourself",
      contents: {
        a!columnsLayout(
          columns: {
            a!columnLayout(
              contents: {
                a!textField(
                  label: "First Name",
                  labelPosition: "ABOVE",
                  saveInto: {},
                  refreshAfter: "UNFOCUS",
                  validations: {}
                )
              }
            ),
            a!columnLayout(
              contents: {
                a!textField(
                  label: "Last Name",
                  labelPosition: "ABOVE",
                  saveInto: {},
                  refreshAfter: "UNFOCUS",
                  validations: {}
                )
              }
            )
          },
          marginBelow: "MORE"
        ),
        a!columnsLayout(
          columns: {
            a!columnLayout(
              contents: {
                a!textField(
                  label: "Email",
                  labelPosition: "ABOVE",
                  saveInto: {},
                  refreshAfter: "UNFOCUS",
                  validations: {}
                )
              }
            ),
            a!columnLayout(
              contents: {
                a!integerField(
                  label: "Phone",
                  labelPosition: "ABOVE",
                  saveInto: {},
                  refreshAfter: "UNFOCUS",
                  validations: {}
                )
              }
            ),
            a!columnLayout(
              contents: {
                a!checkboxField(
                  choiceLabels: {"Email Me", "Text Me"},
                  choiceValues: {1, 2},
                  label: "Contact Preference",
                  labelPosition: "ABOVE",
                  saveInto: {},
                  choiceLayout: "COMPACT",
                  validations: {}
                )
              }
            )
          },
          marginBelow: "MORE"
        ),
        a!textField(
          label: "Street Address",
          labelPosition: "ABOVE",
          saveInto: {},
          refreshAfter: "UNFOCUS",
          validations: {},
          marginBelow: "MORE"
        ),
        a!columnsLayout(
          columns: {
            a!columnLayout(
              contents: {
                a!textField(
                  label: "City",
                  labelPosition: "ABOVE",
                  saveInto: {},
                  refreshAfter: "UNFOCUS",
                  validations: {}
                )
              }
            ),
            a!columnLayout(
              contents: {
                a!dropdownField(
                  choiceLabels: {"Alabama","Alaska","Arizona","Arkansas","California"},
                  choiceValues: {"AL","AK","AZ","AR","CA"},
                  label: "State",
                  labelPosition: "ABOVE",
                  placeholder: "Select state",
                  saveInto: {},
                  searchDisplay: "AUTO",
                  validations: {}
                )
              }
            ),
            a!columnLayout(
              contents: {
                a!integerField(
                  label: "Zip Code",
                  labelPosition: "ABOVE",
                  saveInto: {},
                  refreshAfter: "UNFOCUS",
                  validations: {}
                )
              }
            )
          },
          marginBelow: "MORE"
        )
      }
    ),
    a!wizardStep(
      label: "Coverage Options",
      contents: {}
    ),
    a!wizardStep(
      label: "Quote",
      contents: {}
    )
  },
  contentsWidth: "MEDIUM",
  primaryButtons: {
    a!buttonWidget(
      label: "Submit",
      submit: true,
      style: "SOLID",
      loadingIndicator: true,
      showWhen: fv!isLastStep
    )
  },
  secondaryButtons: {
    a!buttonWidget(
      label: "Cancel",
      value: true,
      saveInto: {},
      submit: true,
      style: "LINK",
      validate: false
    )
  }
)

```

## Core WizardLayout Parameters
- **titleBar**: Text or header template for wizard title
- **steps**: Array of a!wizardStep() definitions
- **style**: Visual style of step indicator (DOT_VERTICAL, CHEVRON_HORIZONTAL, etc.)
- **contentsWidth**: Width constraint for step contents
- **primaryButtons/secondaryButtons**: Additional buttons (Next/Back added automatically)
- **showStepHeadings**: Whether step labels appear above content
- **focusOnFirstInput**: Automatic focus management between steps

## Wizard Styles - Choosing the Right Indicator

### Available Style Options
- **DOT_VERTICAL** (default): Dots on left side, good for many steps
- **DOT_HORIZONTAL**: Dots above content, good for few steps
- **CHEVRON_VERTICAL**: Chevrons on left, good for process flows
- **CHEVRON_HORIZONTAL**: Chevrons above, good for linear workflows
- **LINE_VERTICAL**: Connected lines on left, shows progression
- **LINE_HORIZONTAL**: Connected lines above, compact layout
- **MINIMAL**: No visual indicator, just "Step X of Y" text

### When to Use Each Style
```sail
a!wizardLayout(
  style: "DOT_VERTICAL",        // 4+ steps, tall content
  style: "DOT_HORIZONTAL",      // 2-3 steps, compact content
  style: "CHEVRON_VERTICAL",    // Process with decision points
  style: "CHEVRON_HORIZONTAL",  // Linear workflow, few steps
  style: "LINE_VERTICAL",       // Emphasize step progression
  style: "LINE_HORIZONTAL",     // Minimal space, clear flow
  style: "MINIMAL"              // Mobile-friendly, minimal UI
)
```

## ContentsWidth - Controlling Step Content Width

### Available ContentsWidth Values
- **FULL** (default): Edge-to-edge, uses full container width
- **WIDE**: ~800px maximum width, centered
- **MEDIUM**: ~400px maximum width, centered  
- **NARROW**: ~240px maximum width, centered
- **EXTRA_NARROW**: ~80px maximum width, centered

### Dynamic Defaults by Context
| Context | Default ContentsWidth | Reasoning |
|---------|----------------------|-----------|
| Dialog | FULL | Works with dialog constraints |
| Page | NARROW | Prevents overly wide forms |

## ✅ CORRECT APPROACHES

### Approach 1: Standard Multi-Step Form
```sail
a!wizardLayout(
  titleBar: a!headerTemplateFull(
    title: "Customer Registration",
    secondaryText: "Complete all steps to create account"
  ),
  style: "DOT_VERTICAL",
  contentsWidth: "MEDIUM",
  steps: {
    a!wizardStep(
      label: "Account Details",
      instructions: "Basic account information",
      contents: {
        a!textField(label: "Username", value: local!username, saveInto: local!username),
        a!textField(label: "Email", value: local!email, saveInto: local!email),
        a!encryptedTextField(label: "Password", value: local!password, saveInto: local!password)
      }
    ),
    a!wizardStep(
      label: "Personal Information", 
      contents: {
        a!columnsLayout(
          columns: {
            a!columnLayout(contents: {
              a!textField(label: "First Name", value: local!firstName, saveInto: local!firstName)
            }),
            a!columnLayout(contents: {
              a!textField(label: "Last Name", value: local!lastName, saveInto: local!lastName)
            })
          }
        ),
        a!dateField(label: "Date of Birth", value: local!dob, saveInto: local!dob)
      }
    ),
    a!wizardStep(
      label: "Confirmation",
      contents: {
        a!richTextDisplayField(
          value: "Please review your information and click Submit to complete registration."
        ),
        /* Display summary of entered data */
      }
    )
  },
  primaryButtons: {
    a!buttonWidget(
      label: "Submit",
      submit: true(),
      style: "SOLID",
      showWhen: fv!isLastStep
    )
  },
  secondaryButtons: {
    a!buttonWidget(
      label: "Cancel",
      value: "CANCEL",
      saveInto: local!cancelWizard,
      style: "LINK"
    )
  }
)
```

### Approach 2: Dialog-Optimized Wizard
```sail
a!wizardLayout(
  titleBar: "Quick Setup",
  style: "LINE_HORIZONTAL",      /* ✅ Compact for dialogs */
  contentsWidth: "FULL",         /* ✅ Let dialog control width */
  isTitleBarFixed: true(),       /* ✅ Auto-set in dialogs */
  showButtonDivider: true(),     /* ✅ Auto-set in dialogs */
  steps: {...}
)
```

### Approach 3: Conditional Step Flow
```sail
a!wizardLayout(
  steps: {
    a!wizardStep(
      label: "User Type",
      contents: {
        a!radioButtonField(
          label: "I am a:",
          choiceLabels: {"Individual", "Business"},
          choiceValues: {"INDIVIDUAL", "BUSINESS"},
          value: local!userType,
          saveInto: local!userType
        )
      }
    ),
    a!wizardStep(
      label: "Personal Details",
      showWhen: local!userType = "INDIVIDUAL",  /* ✅ Conditional step */
      contents: {...}
    ),
    a!wizardStep(
      label: "Business Details", 
      showWhen: local!userType = "BUSINESS",    /* ✅ Conditional step */
      contents: {...}
    )
  }
)
```

## Function Variables - Referencing Wizard State

### Available Function Variables
- **fv!activeStepIndex**: Current step number (1-based)
- **fv!isFirstStep**: Boolean, true only on first step
- **fv!isLastStep**: Boolean, true only on last visible step

### Using Function Variables Effectively
```sail
a!wizardStep(
  label: "Review",
  contents: {
    a!richTextDisplayField(
      value: a!richTextItem(
        text: "Step " & fv!activeStepIndex & " of " & length(local!allSteps),
        style: "EMPHASIS"
      )
    ),
    /* Show different content based on step position */
    if(
      fv!isFirstStep,
      "Welcome! Let's get started.",
      if(
        fv!isLastStep,
        "Almost done! Please review your information.",
        "Please continue to the next step."
      )
    )
  },
  /* Disable Next button conditionally */
  disableNextButton: and(
    fv!activeStepIndex = 1,
    a!isNullOrEmpty(local!requiredField)
  )
)
```

## Wizard Step Configuration Best Practices

### Complete WizardStep Structure
```sail
a!wizardStep(
  label: "Step Name",                    /* Required: Shows in indicator */
  instructions: "Helper text",           /* Optional: Shows above content */
  contents: {...},                       /* Required: Step content */
  showWhen: true(),                      /* Optional: Conditional display */
  disableNextButton: false(),            /* Optional: Control navigation */
  validations: {...}                     /* Optional: Step-level validations */
)
```

### Progressive Disclosure Pattern
```sail
/* ✅ GOOD - Show details based on previous choices */
a!wizardStep(
  label: "Product Configuration",
  contents: {
    a!dropdownField(
      label: "Product Type",
      value: local!productType,
      saveInto: local!productType,
      choiceLabels: {"Basic", "Premium", "Enterprise"}
    ),
    /* Show additional options only for Premium/Enterprise */
    if(
      local!productType <> "Basic",
      {
        a!checkboxField(label: "Add-on Features", ...),
        a!integerField(label: "User Licenses", ...)
      },
      {}
    )
  }
)
```

## Decision Matrix: Wizard Style Selection

| Number of Steps | Content Height | Recommended Style | Reasoning |
|-----------------|----------------|-------------------|-----------|
| 2-3 steps | Short | DOT_HORIZONTAL | Clean, doesn't take sidebar space |
| 4-6 steps | Medium | DOT_VERTICAL | Sidebar indicator, room for labels |
| 7+ steps | Any | LINE_VERTICAL | Connected flow, handles many steps |
| Any | Tall/scrolling | VERTICAL styles | Keeps indicator always visible |
| Mobile/minimal | Any | MINIMAL | Responsive, takes least space |

## Common Anti-Patterns to Avoid

### ❌ Using WizardLayout for Single-Step Forms
```sail
/* WRONG - Only one step */
a!wizardLayout(
  steps: {
    a!wizardStep(
      label: "Contact Information",
      contents: {
        a!textField(label: "Name", ...),
        a!textField(label: "Email", ...)
      }
    )
  }
)

/* CORRECT - Use FormLayout instead */
a!formLayout(
  titleBar: "Contact Information",
  contents: {
    a!textField(label: "Name", ...),
    a!textField(label: "Email", ...)
  }
)
```

### ❌ Overriding Automatic Button Behavior
```sail
/* WRONG - Trying to configure Next/Back buttons */
a!wizardLayout(
  primaryButtons: {
    a!buttonWidget(label: "Next", ...), /* ❌ Next is automatic */
    a!buttonWidget(label: "Submit", submit: true())
  },
  secondaryButtons: {
    a!buttonWidget(label: "Back", ...), /* ❌ Back is automatic */
    a!buttonWidget(label: "Cancel", ...)
  }
)

/* CORRECT - Only add additional buttons */
a!wizardLayout(
  primaryButtons: {
    a!buttonWidget(                      /* ✅ Additional primary button */
      label: "Save Draft",
      saveInto: local!saveDraft
    ),
    a!buttonWidget(                      /* ✅ Submit only on last step */
      label: "Submit",
      submit: true(),
      showWhen: fv!isLastStep
    )
  },
  secondaryButtons: {
    a!buttonWidget(label: "Cancel", ...) /* ✅ Additional secondary button */
  }
)
```

### ❌ Inconsistent Step Validation Patterns
```sail
/* WRONG - Mixed validation approaches */
a!wizardStep(
  contents: {
    a!textField(
      validations: if(isnull(local!name), "Name is required", null) /* ❌ Inconsistent */
    )
  },
  disableNextButton: isnull(local!email) /* ❌ Different field validation */
)

/* CORRECT - Consistent validation strategy */
a!wizardStep(
  contents: {
    a!textField(
      value: local!name,
      saveInto: local!name,
      required: true()                   /* ✅ Use built-in validation */
    ),
    a!textField(
      value: local!email, 
      saveInto: local!email,
      required: true()                   /* ✅ Consistent approach */
    )
  },
  disableNextButton: or(              /* ✅ Clear disable logic */
    a!isNullOrEmpty(local!name),
    a!isNullOrEmpty(local!email)
  )
)
```

## Quick Reference: Layout Selection Guide

**"Do I have multiple logical steps for data collection?"**
- Yes → **WizardLayout**

**"Is this a single form, even if complex?"**
- Yes → **FormLayout**

**"Do I need independent scrolling areas?"**
- Yes → **PaneLayout**

**"Am I primarily displaying information?"**
- Yes → **HeaderContentLayout**

### Wizard vs Form Decision Tree
```
Is user input split into logical steps?
├─ Yes: Are there 2+ distinct phases?
│  ├─ Yes → WizardLayout
│  └─ No → FormLayout (with sections)
└─ No → FormLayout
```

## Advanced Patterns

### Pattern 1: Data Collection with Summary
```sail
a!wizardLayout(
  steps: {
    /* Data collection steps */
    a!wizardStep(label: "Basic Info", contents: {...}),
    a!wizardStep(label: "Preferences", contents: {...}),
    
    /* Summary step with review */
    a!wizardStep(
      label: "Review",
      contents: {
        a!sectionLayout(
          label: "Please review your information",
          contents: {
            a!richTextDisplayField(
              value: {
                a!richTextItem(text: "Name: ", style: "STRONG"),
                local!firstName & " " & local!lastName,
                char(10),
                a!richTextItem(text: "Email: ", style: "STRONG"), 
                local!email
              }
            )
          }
        )
      }
    )
  }
)
```

### Pattern 2: Conditional Navigation
```sail
a!wizardStep(
  label: "Account Type",
  disableNextButton: a!isNullOrEmpty(local!accountType),
  contents: {
    a!radioButtonField(
      label: "Select account type:",
      value: local!accountType,
      saveInto: {
        local!accountType,
        /* Reset dependent values when changing */
        a!save(local!businessName, null),
        a!save(local!taxId, null)
      }
    )
  }
)
```

Remember: WizardLayout automatically handles step navigation, progress indication, and button management. Focus on organizing your content into logical steps and let the wizard handle the navigation flow.