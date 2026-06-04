# Card Group List Items

Examples of how to lay out different types of list item as cards

## User List Cards

```sail
a!localVariables(
  /* Sample user data for mockup */
  local!users: {
    a!map(
      name: "Sarah Johnson",
      initials: "SJ",
      role: "Product Manager",
      department: "Engineering",
      departmentColor: "#3B82F6",
      avatarBg: "#3B82F6",
      email: "sarah.johnson@company.com",
      lastActive: "Active now"
    ),
    a!map(
      name: "Michael Chen",
      initials: "MC",
      role: "Senior Developer",
      department: "Engineering",
      departmentColor: "#3B82F6",
      avatarBg: "#8B5CF6",
      email: "michael.chen@company.com",
      lastActive: "2 hours ago"
    ),
    a!map(
      name: "Emily Rodriguez",
      initials: "ER",
      role: "UX Designer",
      department: "Design",
      departmentColor: "#EC4899",
      avatarBg: "#EC4899",
      email: "emily.rodriguez@company.com",
      lastActive: "Yesterday"
    )
  },

  a!cardGroupLayout(
    cards: a!forEach(
      items: local!users,
      expression: a!cardLayout(
        contents: {
          a!sideBySideLayout(
            items: {
              a!sideBySideItem(
                item: a!stampField(
                  text: fv!item.initials,
                  backgroundColor: fv!item.avatarBg,
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
                      size: "MEDIUM",
                      color: "#262626"
                    ),
                    char(10),
                    a!richTextItem(
                      text: fv!item.role,
                      color: "#6B7280",
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
                    text: fv!item.department,
                    backgroundColor: fv!item.departmentColor
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
        padding: "STANDARD",
        shape: "ROUNDED",
        link: a!dynamicLink(
          value: fv!item,
          saveInto: {
            /* Navigate to user profile or show details */
          }
        ),
        showShadow: false
      )
    ),
    cardWidth: "NARROW_PLUS",
    spacing: "DENSE"
  )
)
```

## Task List Cards

```sail
a!localVariables(
  /* Sample task data for mockup */
  local!tasks: {
    a!map(
      title: "Review Q4 Budget Proposal",
      description: "Final review needed before board meeting",
      assignee: "Sarah Johnson",
      priority: "High",
      priorityColor: "#DC2626",
      priorityIcon: "exclamation-circle",
      dueDate: "Today",
      dueDateColor: "#DC2626",
      status: "In Progress",
      statusColor: "#3B82F6",
      taskIcon: "file-text",
      taskIconBg: "#FEE2E2",
      taskIconColor: "#DC2626"
    ),
    a!map(
      title: "Update Customer Dashboard",
      description: "Add new analytics widgets and filters",
      assignee: "Michael Chen",
      priority: "Medium",
      priorityColor: "#F59E0B",
      priorityIcon: "minus-circle",
      dueDate: "Tomorrow",
      dueDateColor: "#F59E0B",
      status: "To Do",
      statusColor: "#6B7280",
      taskIcon: "code",
      taskIconBg: "#FEF3C7",
      taskIconColor: "#F59E0B"
    ),
    a!map(
      title: "Design System Documentation",
      description: "Document component usage guidelines",
      assignee: "Emily Rodriguez",
      priority: "Low",
      priorityColor: "#059669",
      priorityIcon: "arrow-down",
      dueDate: "Next Week",
      dueDateColor: "#6B7280",
      status: "Completed",
      statusColor: "#059669",
      taskIcon: "paint-brush",
      taskIconBg: "#D1FAE5",
      taskIconColor: "#059669"
    )
  },

  a!cardGroupLayout(
    cards: a!forEach(
      items: local!tasks,
      expression: a!cardLayout(
        contents: {
          a!sideBySideLayout(
            items: {
              a!sideBySideItem(
                item: a!stampField(
                  icon: fv!item.taskIcon,
                  backgroundColor: fv!item.taskIconBg,
                  contentColor: fv!item.taskIconColor,
                  size: "SMALL",
                  shape: "SEMI_ROUNDED",
                  labelPosition: "COLLAPSED"
                ),
                width: "MINIMIZE"
              ),
              a!sideBySideItem(
                item: a!richTextDisplayField(
                  value: {
                    a!richTextItem(
                      text: fv!item.title,
                      style: "STRONG",
                      size: "MEDIUM",
                      color: "#262626"
                    ),
                    char(10),
                    a!richTextItem(
                      text: fv!item.description,
                      color: "#6B7280",
                      size: "SMALL"
                    ),
                    char(10),
                    a!richTextIcon(
                      icon: "user",
                      color: "#9CA3AF",
                      size: "SMALL"
                    ),
                    " ",
                    a!richTextItem(
                      text: fv!item.assignee,
                      color: "#6B7280",
                      size: "SMALL"
                    )
                  },
                  labelPosition: "COLLAPSED"
                ),
                width: "AUTO"
              ),
              a!sideBySideItem(
                item: a!richTextDisplayField(
                  value: {
                    a!richTextItem(
                      text: fv!item.status,
                      color: fv!item.statusColor,
                      size: "SMALL",
                      style: "STRONG"
                    ),
                    char(10),
                    a!richTextIcon(
                      icon: "calendar",
                      color: fv!item.dueDateColor,
                      size: "SMALL"
                    ),
                    " ",
                    a!richTextItem(
                      text: fv!item.dueDate,
                      color: fv!item.dueDateColor,
                      size: "SMALL"
                    ),
                    char(10),
                    a!richTextIcon(
                      icon: fv!item.priorityIcon,
                      color: fv!item.priorityColor,
                      size: "SMALL"
                    ),
                    " ",
                    a!richTextItem(
                      text: fv!item.priority,
                      color: fv!item.priorityColor,
                      size: "SMALL"
                    )
                  },
                  labelPosition: "COLLAPSED",
                  align: "RIGHT"
                ),
                width: "MINIMIZE"
              )
            },
            alignVertical: "TOP",
            spacing: "SPARSE"
          )
        },
        style: "#FFFFFF",
        showBorder: true,
        padding: "STANDARD",
        shape: "ROUNDED",
        link: a!dynamicLink(
          value: fv!item,
          saveInto: {
            /* Navigate to task details */
          }
        ),
        showShadow: false
      )
    ),
    cardWidth: "MEDIUM_PLUS",
    spacing: "DENSE"
  )
)
```

## Message List Cards

```sail
a!localVariables(
  /* Sample message data for mockup */
  local!messages: {
    a!map(
      sender: "Sarah Johnson",
      initials: "SJ",
      message: "Hi team, I've attached the final Q4 budget proposal for your review. Please let me know if you have any questions...",
      timestamp: "10 min ago",
      unread: true,
      avatarBg: "#3B82F6"
    ),
    a!map(
      sender: "Michael Chen",
      initials: "MC",
      message: "I've completed the dashboard updates. Can you review the pull request when you have a moment? Thanks!",
      timestamp: "2 hrs ago",
      unread: true,
      avatarBg: "#8B5CF6"
    ),
    a!map(
      sender: "Emily Rodriguez",
      initials: "ER",
      message: "The new component library is ready. I've updated all the documentation and examples in Figma...",
      timestamp: "Yesterday",
      unread: false,
      avatarBg: "#EC4899"
    )
  },

  a!cardGroupLayout(
    cards: a!forEach(
      items: local!messages,
      expression: a!cardLayout(
        contents: {
          a!sideBySideLayout(
            items: {
              a!sideBySideItem(
                item: a!stampField(
                  text: fv!item.initials,
                  backgroundColor: fv!item.avatarBg,
                  contentColor: "#FFFFFF",
                  size: "TINY",
                  shape: "SEMI_ROUNDED",
                  labelPosition: "COLLAPSED"
                ),
                width: "MINIMIZE"
              ),
              a!sideBySideItem(
                item: a!richTextDisplayField(
                  value: {
                    a!richTextItem(
                      text: fv!item.sender,
                      style: if(fv!item.unread, "STRONG", "PLAIN"),
                      size: "STANDARD",
                      color: "#262626"
                    ),
                    char(10),
                    a!richTextItem(
                      text: fv!item.message,
                      color: "#6B7280",
                      size: "STANDARD"
                    )
                  },
                  labelPosition: "COLLAPSED"
                ),
                width: "AUTO"
              ),
              a!sideBySideItem(
                item: a!richTextDisplayField(
                  value: {
                    a!richTextItem(
                      text: fv!item.timestamp,
                      color: "#9CA3AF",
                      size: "SMALL"
                    )
                  },
                  labelPosition: "COLLAPSED",
                  align: "RIGHT"
                ),
                width: "MINIMIZE"
              )
            },
            alignVertical: "TOP",
            spacing: "STANDARD"
          )
        },
        style: if(fv!item.unread, "#F9FAFB", "#FFFFFF"),
        showBorder: true,
        padding: "STANDARD",
        shape: "ROUNDED",
        link: a!dynamicLink(
          value: fv!item,
          saveInto: {
            /* Open message details */
          }
        ),
        showShadow: false
      )
    ),
    cardWidth: "WIDE_PLUS",
    spacing: "DENSE"
  )
)
```

## Photo Gallery List Cards

For displaying items with prominent images (real estate, products, media, etc.):

```sail
a!localVariables(
  /* Sample property data for mockup */
  local!properties: {
    a!map(
      image: "https://images.unsplash.com/photo-1512917774080-9991f1c4c75?w=800",
      price: "$1,695,000",
      beds: 3,
      baths: 2.5,
      sqft: "2,403",
      address: "12345 Maple Ave, Palm Springs, CA 92262",
      daysListed: 2,
      status: "NEW LISTING",
      statusColor: "#FF9900"
    ),
    a!map(
      image: "https://images.unsplash.com/photo-1575517171740-99871f6c4c75?w=800",
      price: "$2,150,000",
      beds: 4,
      baths: 3.5,
      sqft: "2,942",
      address: "2345 Mesa Blvd, Palm Springs, CA 92264",
      daysListed: 15,
      status: "OPEN HOUSE SCHEDULED",
      statusColor: "#38761D"
    ),
    a!map(
      image: "https://images.unsplash.com/photo-1582268611958-ebfd16ef9cdb?w=800",
      price: "$1,945,000",
      beds: 3,
      baths: 2.5,
      sqft: "2,178",
      address: "345 Main St, Cathedral City, CA 92234",
      daysListed: 26,
      status: "OPEN HOUSE SCHEDULED",
      statusColor: "#38761D"
    )
  },

  a!cardGroupLayout(
    cards: a!forEach(
      items: local!properties,
      expression: a!cardLayout(
        contents: {
          a!billboardLayout( /* Use a billboard layout for each item photo because it guarantees the same height for all images even if the source photos have different aspect ratios */
            backgroundMedia: a!webImage(source: fv!item.image),
            backgroundColor: "#F0F0F0",
            height: "SHORT_PLUS",
            marginBelow: "NONE",
            overlay: a!fullOverlay( /* Using a billboard overlay allows information, such as a tag or rich text, to be superimposed upon the photo */
              alignVertical: "TOP",
              contents: {
                a!tagField(
                  labelPosition: "COLLAPSED",
                  tags: a!tagItem(
                    text: fv!item.status,
                    backgroundColor: fv!item.statusColor
                  )
                )
              },
              style: "NONE"
            )
          ),
          a!cardLayout(
            contents: {
              a!sideBySideLayout(
                items: {
                  a!sideBySideItem(
                    item: a!richTextDisplayField(
                      labelPosition: "COLLAPSED",
                      value: {
                        a!richTextItem(
                          text: fv!item.price,
                          size: "MEDIUM_PLUS",
                          style: "STRONG"
                        )
                      }
                    )
                  ),
                  a!sideBySideItem(
                    item: a!richTextDisplayField(
                      labelPosition: "COLLAPSED",
                      value: {
                        a!richTextItem(
                          text: {
                            a!richTextIcon(icon: "calendar"),
                            " " & fv!item.daysListed & "d"
                          },
                          color: "SECONDARY",
                          size: "MEDIUM"
                        )
                      }
                    ),
                    width: "MINIMIZE"
                  )
                },
                alignVertical: "MIDDLE",
                marginBelow: "STANDARD"
              ),
              a!richTextDisplayField(
                labelPosition: "COLLAPSED",
                value: {
                  a!richTextItem(
                    text: fv!item.beds & " Beds  ",
                    size: "STANDARD"
                  ),
                  " • ",
                  fv!item.baths & " Baths",
                  " • ",
                  fv!item.sqft & " Sq. Ft.",
                  char(10),
                  a!richTextItem(
                    text: fv!item.address,
                    size: "SMALL",
                    color: "#6B7280"
                  )
                },
                preventWrapping: false
              )
            },
            height: "AUTO",
            style: "NONE",
            padding: "STANDARD",
            marginBelow: "NONE",
            showBorder: false
          )
        },
        link: a!dynamicLink(
          value: fv!item,
          saveInto: {
            /* Navigate to property details */
          }
        ),
        height: "AUTO",
        style: "NONE",
        shape: "SEMI_ROUNDED",
        padding: "NONE",
        showBorder: true
      )
    ),
    cardWidth: "MEDIUM",
    spacing: "STANDARD"
  )
)
```

## Pattern Variations

### Task List with Progress Bar
For tasks that track completion percentage:

```sail
a!cardLayout(
  contents: {
    a!cardLayout(
      contents: {
        a!sideBySideLayout(
          items: {
            a!sideBySideItem(
              item: a!stampField(
                icon: "tasks",
                backgroundColor: "#EFF6FF",
                contentColor: "#3B82F6",
                size: "SMALL",
                shape: "SEMI_ROUNDED",
                labelPosition: "COLLAPSED"
              ),
              width: "MINIMIZE"
            ),
            a!sideBySideItem(
              item: a!richTextDisplayField(
                value: {
                  a!richTextItem(
                    text: "Website Redesign Project",
                    style: "STRONG",
                    size: "MEDIUM",
                    color: "#262626"
                  ),
                  char(10),
                  a!richTextItem(
                    text: "12 of 18 tasks completed",
                    color: "#6B7280",
                    size: "SMALL"
                  )
                },
                labelPosition: "COLLAPSED"
              ),
              width: "AUTO"
            ),
            a!sideBySideItem(
              item: a!richTextDisplayField(
                value: {
                  a!richTextItem(
                    text: "67%",
                    style: "STRONG",
                    size: "MEDIUM",
                    color: "#3B82F6"
                  )
                },
                labelPosition: "COLLAPSED",
                align: "RIGHT"
              ),
              width: "MINIMIZE"
            )
          },
          alignVertical: "MIDDLE",
          spacing: "STANDARD"
        )
      },
      showBorder: false,
      padding: "STANDARD"
    ),
    a!progressBarField(
      percentage: 67,
      color: "#3B82F6",
      style: "THIN",
      showPercentage: false,
      labelPosition: "COLLAPSED",
      marginBelow: "NONE"
    )
  },
  style: "#FFFFFF",
  showBorder: true,
  padding: "NONE",
  shape: "ROUNDED"
)
```

### Message List with Action Buttons
For messages that need quick actions:

```sail
a!cardLayout(
  contents: {
    a!sideBySideLayout(
      items: {
        a!sideBySideItem(
          item: a!stampField(
            icon: "envelope",
            backgroundColor: "#3B82F6",
            contentColor: "#FFFFFF",
            size: "SMALL",
            shape: "ROUNDED",
            labelPosition: "COLLAPSED"
          ),
          width: "MINIMIZE"
        ),
        a!sideBySideItem(
          item: a!richTextDisplayField(
            value: {
              a!richTextItem(
                text: "Project Update Required",
                style: "STRONG",
                size: "MEDIUM",
                color: "#262626"
              ),
              char(10),
              a!richTextItem(
                text: "Please provide status update for Q4 milestones...",
                color: "#6B7280"
              ),
              char(10),
              a!richTextItem(
                text: "5 minutes ago",
                color: "#9CA3AF",
                size: "SMALL"
              )
            },
            labelPosition: "COLLAPSED"
          ),
          width: "AUTO"
        ),
        a!sideBySideItem(
          item: a!buttonArrayLayout(
            buttons: {
              a!buttonWidget(
                label: "Reply",
                size: "SMALL",
                style: "OUTLINE",
                color: "#3B82F6"
              )
            },
            marginBelow: "NONE",
            align: "END"
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
  padding: "STANDARD",
  shape: "ROUNDED"
)
```
