# KPI Cards

Examples of how to lay out KPI cards

```sail
a!localVariables(
  /* Sample KPI data for mockup */
  local!currentRevenue: 2456789,
  local!revenueGrowth: 12.5,
  local!totalUsers: 15847,
  local!userGrowth: 8.3,
  local!completedTasks: 324,
  local!totalTasks: 378,

  a!cardGroupLayout(
    cards: {
      /* Revenue KPI Card */
      a!cardLayout(
        contents: {
          a!sideBySideLayout(
            items: {
              a!sideBySideItem(
                item: a!stampField(
                  icon: "dollar",
                  backgroundColor: "#2C3E50",
                  contentColor: "#FFFFFF",
                  size: "TINY",
                  shape: "ROUNDED", /* Show decorative stamp as circle shape */
                  labelPosition: "COLLAPSED"
                ),
                width: "MINIMIZE"
              ),
              a!sideBySideItem(
                item: a!richTextDisplayField(
                  value: {
                    a!richTextItem(
                      text: "Total Revenue",
                      color: "#6B7280",
                      size: "STANDARD"
                    ),
                    char(10),
                    a!richTextItem(
                      text: dollar(local!currentRevenue,0), /* Drop decimals from whole dollar amount */
                      style: "STRONG",
                      size: "LARGE",
                      color: "#262626"
                    ),
                    char(10),
                    a!richTextIcon(
                      icon: "arrow-up",
                      color: "#7F8C8D",
                      size: "SMALL"
                    ),
                    " ",
                    a!richTextItem(
                      text: local!revenueGrowth & "% from last quarter",
                      color: "#7F8C8D",
                      size: "SMALL",
                      style: "STRONG"
                    )
                  },
                  labelPosition: "COLLAPSED"
                ),
                width: "AUTO"
              )
            },
            alignVertical: "MIDDLE",
            spacing: "STANDARD"
          )
        },
        style: "#FFFFFF",
        showBorder: true,
        padding: "MORE",
        shape: "ROUNDED"
      ),

      /* Active Users KPI Card */
      a!cardLayout(
        contents: {
          a!sideBySideLayout(
            items: {
              a!sideBySideItem(
                item: a!stampField(
                  icon: "users",
                  backgroundColor: "#34495E",
                  contentColor: "#FFFFFF",
                  size: "TINY",
                  shape: "SEMI_ROUNDED", /* Show decorative stamp as square with slight corner rounding */
                  labelPosition: "COLLAPSED"
                ),
                width: "MINIMIZE"
              ),
              a!sideBySideItem(
                item: a!richTextDisplayField(
                  value: {
                    a!richTextItem(
                      text: "Active Users",
                      color: "#6B7280",
                      size: "STANDARD"
                    ),
                    char(10),
                    a!richTextItem(
                      text: text(local!totalUsers, "##,###"),
                      style: "STRONG",
                      size: "LARGE",
                      color: "#262626"
                    ),
                    char(10),
                    a!richTextIcon(
                      icon: "arrow-up",
                      color: "#7F8C8D",
                      size: "SMALL"
                    ),
                    " ",
                    a!richTextItem(
                      text: local!userGrowth & "% this month",
                      color: "#7F8C8D",
                      size: "SMALL",
                      style: "STRONG"
                    )
                  },
                  labelPosition: "COLLAPSED"
                ),
                width: "AUTO"
              )
            },
            alignVertical: "MIDDLE",
            spacing: "STANDARD"
          )
        },
        style: "#FFFFFF",
        showBorder: true,
        padding: "MORE",
        shape: "ROUNDED"
      ),

      /* Task Completion KPI Card With Prominent Progress Bar */
      a!cardLayout(
        contents: {
          a!cardLayout( /* Uses a nested, borderless card to add padding around text labels and values */
            contents: {
              a!sideBySideLayout(
                items: {
                  a!sideBySideItem(
                    item: a!stampField(
                      icon: "check-circle",
                      backgroundColor: "#7F8C8D",
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
                          text: "Tasks Completed",
                          color: "#6B7280",
                          size: "STANDARD"
                        ),
                        char(10),
                        a!richTextItem(
                          text: local!completedTasks & " / " & local!totalTasks,
                          style: "STRONG",
                          size: "LARGE",
                          color: "#262626"
                        ),
                        char(10),
                        a!richTextItem(
                          text: fixed(local!completedTasks / local!totalTasks * 100, 1) & "% completion rate",
                          color: "#7F8C8D",
                          size: "SMALL",
                          style: "STRONG"
                        )
                      },
                      labelPosition: "COLLAPSED"
                    ),
                    width: "AUTO"
                  )
                },
                alignVertical: "MIDDLE",
                spacing: "STANDARD"
              ),   
            },
            showBorder: false,
            padding: "MORE"
          ),
          a!progressBarField( /* A flush progress bar for visual interest and to emphasize the progress towards target */
            percentage: tointeger(local!completedTasks / local!totalTasks * 100),
            color: "#7F8C8D",
            style: "THICK",
            showPercentage: false,
            labelPosition: "COLLAPSED",
            marginBelow: "NONE"
          )
        },
        style: "#FFFFFF",
        showBorder: true,
        padding: "NONE", /* No padding on the outer KPI card to allow the progress bar to be flush with the left/right/bottom */
        shape: "ROUNDED"
      ),

      /* Conversion Rate KPI Card with Gauge */
      a!cardLayout(
        contents: {
          a!sideBySideLayout(
            items: {
              a!sideBySideItem(
                item: a!gaugeField( /* Use of gauge field to visually emphasize percentage achievement */
                  percentage: 73.2,
                  primaryText: "73%",
                  secondaryText: "Rate",
                  color: "#95A5A6",
                  size: "SMALL",
                  labelPosition: "COLLAPSED"
                ),
                width: "MINIMIZE"
              ),
              a!sideBySideItem(
                item: a!richTextDisplayField(
                  value: {
                    a!richTextItem(
                      text: "Conversion Rate",
                      color: "#6B7280",
                      size: "STANDARD"
                    ),
                    char(10),
                    a!richTextItem(
                      text: "Lead to Customer",
                      style: "STRONG",
                      size: "MEDIUM_PLUS",
                      color: "#262626"
                    ),
                    char(10),
                    a!richTextIcon(
                      icon: "arrow-up",
                      color: "#7F8C8D",
                      size: "SMALL"
                    ),
                    " ",
                    a!richTextItem(
                      text: "5.2% vs last month",
                      color: "#7F8C8D",
                      size: "SMALL",
                      style: "STRONG"
                    )
                  },
                  labelPosition: "COLLAPSED"
                ),
                width: "AUTO"
              )
            },
            alignVertical: "MIDDLE",
            spacing: "STANDARD"
          )
        },
        style: "#FFFFFF",
        showBorder: true,
        padding: "MORE",
        shape: "ROUNDED"
      ),

      /* Customer Satisfaction KPI Card */
      a!cardLayout(
        contents: {
          a!sideBySideLayout(
            items: {
              a!sideBySideItem(
                item: a!stampField(
                  icon: "star",
                  backgroundColor: "#2C3E50",
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
                      text: "Customer Satisfaction",
                      color: "#6B7280",
                      size: "STANDARD"
                    ),
                    char(10),
                    a!richTextItem(
                      text: "4.8 / 5.0",
                      style: "STRONG",
                      size: "LARGE",
                      color: "#262626"
                    ),
                    char(10),
                    a!richTextItem(
                      text: "Based on 2,847 reviews",
                      color: "#6B7280",
                      size: "SMALL"
                    )
                  },
                  labelPosition: "COLLAPSED"
                ),
                width: "AUTO"
              ),
              a!sideBySideItem(
                item: a!tagField( /* Use of tag field to show an extra status flag or qualitative assessment */
                  tags: a!tagItem(
                    text: "EXCELLENT",
                    backgroundColor: "#2C3E50"
                  ),
                  size: "SMALL",
                  labelPosition: "COLLAPSED"
                ),
                width: "MINIMIZE"
              )
            },
            alignVertical: "MIDDLE",
            spacing: "STANDARD"
          )
        },
        style: "#FFFFFF",
        showBorder: true,
        padding: "MORE",
        shape: "ROUNDED"
      )
    },
    cardWidth: "NARROW_PLUS",
    spacing: "STANDARD"
  )
)
```