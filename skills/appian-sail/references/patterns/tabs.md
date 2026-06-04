# Tabs

## Prefer `a!tabLayout()` for Standard Tabs

For standard tab navigation, use the native `a!tabLayout()` component. See `guidelines/layouts/tab-layout-instructions.md` for usage, parameters, and patterns.

The patterns below are for **custom tab styling only** — use them when you need badges, counts on tab labels, tabs integrated into a colored page header, or custom decorative bar styling that the native component doesn't support.

---

## Custom Tab Patterns

### Basic Horizontal Tab Bar
```sail
a!localVariables(
  local!tabs: { "Overview", "Details", "History", "Settings" },
  local!selectedTab: 1,
  local!backgroundColor: "#FFFFFF", /* Set this to the background color of the page or tab bar container - used to make the decorative bar invisible on non-selected tabs since TRANSPARENT isn't available */
  a!cardLayout(
    contents: {
      a!cardLayout(
        contents: {
          a!columnsLayout(
            columns: {
              a!forEach(
                items: local!tabs,
                expression: {
                  a!columnLayout(
                    contents: {
                      a!cardLayout(
                        contents: {
                          a!richTextDisplayField(
                            labelPosition: "COLLAPSED",
                            value: {
                              a!richTextItem(
                                text: { fv!item },
                                color: "STANDARD",
                                size: "STANDARD",
                                style: if(
                                  fv!index = local!selectedTab,
                                  "STRONG",
                                  "PLAIN"
                                )
                              )
                            },
                            align: "CENTER",
                            marginAbove: "STANDARD",
                            marginBelow: "LESS"
                          )
                        },
                        link: if(
                          fv!index = local!selectedTab,
                          {},
                          a!dynamicLink(
                            value: fv!index,
                            saveInto: local!selectedTab
                          )
                        ),
                        height: "AUTO",
                        style: "NONE",
                        padding: "EVEN_LESS",
                        marginBelow: "NONE",
                        showBorder: false,
                        decorativeBarPosition: "BOTTOM",
                        decorativeBarColor: if(
                          fv!index = local!selectedTab,
                          "ACCENT",
                          local!backgroundColor
                        ),
                        accessibilityText: if(
                          fv!index = local!selectedTab,
                          "Tab Selected",
                          "Tab"
                        )
                      )
                    },
                    width: "NARROW"
                  )
                }
              ),
              a!columnLayout( /* Empty filler column to comply with best practice of always having an AUTO column */
                width: "AUTO"
              )
            },
            marginBelow: "NONE",
            spacing: "NONE",
            showDividers: false
          )
        },
        height: "AUTO",
        style: "NONE",
        padding: "NONE",
        marginBelow: "NONE",
        showBorder: false,
        showShadow: false
      ),
      a!cardLayout(
        contents: {
          /* Selected tab contents here */
        },
        height: "AUTO",
        style: "NONE",
        padding: "STANDARD",
        marginBelow: "NONE",
        showBorder: false
      )
    },
    height: "AUTO",
    style: "TRANSPARENT",
    padding: "NONE",
    marginBelow: "STANDARD",
    showBorder: false
  )
)
```

## Horizontal Tab Bar with Optional Badges
```sail
a!localVariables(
  local!tabs: {
    a!map(title: "Applied Clauses", badge: "5"),
    a!map(title: "Questionnaire", badge: "2"),
    a!map(title: "Excluded Clauses", badge: "1"),
    a!map(title: "Log History", badge: "0")
  },
  local!selectedTab: 1,
  local!backgroundColor: "#FFFFFF", /* Set this to the background color of the page or tab bar container - used to make the decorative bar invisible on non-selected tabs since TRANSPARENT isn't available */
  a!cardLayout(
    contents: {
      a!cardLayout(
        contents: {
          a!columnsLayout(
            columns: {
              a!forEach(
                items: local!tabs,
                expression: {
                  a!columnLayout(
                    contents: {
                      a!cardLayout(
                        contents: {
                          a!sideBySideLayout(
                            items: {
                              a!sideBySideItem(),
                              a!sideBySideItem(
                                /* Tab label */
                                item: a!richTextDisplayField(
                                  labelPosition: "COLLAPSED",
                                  value: {
                                    a!richTextItem(
                                      text: { fv!item.title },
                                      color: "STANDARD",
                                      size: "STANDARD",
                                      style: if(
                                        fv!index = local!selectedTab,
                                        "STRONG",
                                        "PLAIN"
                                      )
                                    )
                                  },
                                  align: "CENTER",
                                  marginBelow: "NONE"
                                ),
                                width: "MINIMIZE"
                              ),
                              /* Conditionally show badge when value > 0 */
                              if(
                                tointeger(fv!item.badge) > 0,
                                a!sideBySideItem(
                                  /* Optional badge */
                                  item: a!tagField(
                                    labelPosition: "COLLAPSED",
                                    tags: {
                                      a!tagItem(
                                        text: fv!item.badge,
                                        backgroundColor: "#EDEEF2",
                                        textColor: "#2E2E35"
                                      )
                                    },
                                    size: "SMALL",
                                    marginBelow: "NONE"
                                  ),
                                  width: "MINIMIZE"
                                ),
                                {}
                              ),
                              a!sideBySideItem()
                            },
                            alignVertical: "BOTTOM",
                            spacing: "",
                            marginAbove: "STANDARD",
                            marginBelow: "LESS"
                          )
                        },
                        link: if(
                          fv!index = local!selectedTab,
                          {},
                          a!dynamicLink(
                            value: fv!index,
                            saveInto: local!selectedTab
                          )
                        ),
                        height: "AUTO",
                        style: "NONE",
                        padding: "EVEN_LESS",
                        marginBelow: "NONE",
                        showBorder: false,
                        decorativeBarPosition: "BOTTOM",
                        decorativeBarColor: if(
                          fv!index = local!selectedTab,
                          "ACCENT",
                          local!backgroundColor
                        ),
                        accessibilityText: if(
                          fv!index = local!selectedTab,
                          "Tab Selected",
                          "Tab"
                        )
                      )
                    },
                    width: "NARROW"
                  )
                }
              ),
              a!columnLayout( /* Empty filler column to comply with best practice of always having an AUTO column */
                width: "AUTO"
              )
            },
            marginBelow: "NONE",
            spacing: "NONE",
            showDividers: false
          )
        },
        height: "AUTO",
        style: "NONE",
        padding: "NONE",
        marginBelow: "NONE",
        showBorder: false,
        showShadow: false
      ),
      a!cardLayout(
        contents: {
          /* Selected tab contents here */
        },
        height: "AUTO",
        style: "NONE",
        padding: "STANDARD",
        marginBelow: "NONE",
        showBorder: false
      )
    },
    height: "AUTO",
    style: "TRANSPARENT",
    padding: "NONE",
    marginBelow: "STANDARD",
    showBorder: false
  )
)
```
## Horizontal Tab Bar Integrated into Page Header

Use this pattern to incorporate tabs into the page header. Don't use if instructed to place tabs somewhere else on the page.

```sail
a!localVariables(
  local!tabs: {
    "Overview",
    "Details",
    "History",
    "Settings"
  },
  local!selectedTab: 1,
  local!headerBackgroundColor: "#1C2C44",
  a!headerContentLayout(
    header: {
      a!cardLayout(
        contents: {
          a!cardLayout(
            /* Page title and other header content wrapped in a borderless card to provide padding */
            contents: {
              a!headingField(
                text: "Project Dashboard",
                size: "LARGE",
                headingTag: "H1",
                fontWeight: "BOLD",
                color: "#FFFFFF",
                marginBelow: "STANDARD"
              )
            },
            style: "TRANSPARENT",
            padding: "MORE",
            showBorder: false,
            marginBelow: "NONE"
          ),
          a!columnsLayout(
            /* Tab bar */
            columns: {
              a!forEach(
                items: local!tabs,
                expression: {
                  a!columnLayout(
                    contents: {
                      a!cardLayout(
                        contents: {
                          a!richTextDisplayField(
                            labelPosition: "COLLAPSED",
                            value: {
                              a!richTextItem(
                                text: { fv!item },
                                color: "#FFFFFF",
                                size: "STANDARD",
                                style: if(
                                  fv!index = local!selectedTab,
                                  "STRONG",
                                  "PLAIN"
                                )
                              )
                            },
                            align: "CENTER",
                            marginAbove: "STANDARD",
                            marginBelow: "LESS"
                          )
                        },
                        link: if(
                          fv!index = local!selectedTab,
                          {},
                          a!dynamicLink(
                            value: fv!index,
                            saveInto: local!selectedTab
                          )
                        ),
                        height: "AUTO",
                        style: "TRANSPARENT",
                        padding: "EVEN_LESS",
                        marginBelow: "NONE",
                        showBorder: false,
                        decorativeBarPosition: "BOTTOM",
                        decorativeBarColor: if(
                          fv!index = local!selectedTab,
                          "#ffffff",
                          /* On a dark header background, use white to highlight selected tab */
                          local!headerBackgroundColor
                        ),
                        accessibilityText: if(
                          fv!index = local!selectedTab,
                          "Tab Selected",
                          "Tab"
                        )
                      )
                    },
                    width: "NARROW"
                  )
                }
              ),
              a!columnLayout(
                /* Empty filler column to comply with best practice of always having an AUTO column */
                width: "AUTO"
              )
            },
            marginBelow: "NONE",
            spacing: "NONE",
            showDividers: false
          )
        },
        height: "AUTO",
        style: "#1C2C44",
        padding: "EVEN_LESS",
        /* Reduced header padding to allow tab bar to appear more flush with edges of header */
        marginBelow: "NONE",
        showBorder: false
      )
    },
    contents: {
      /* Selected tab contents here */

    },
    backgroundColor: "#F5F6F8"
  )
)
```
