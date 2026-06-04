# Message Banners

Examples of how to show informational, warning, etc. messages in cards

## Basic Messages
```sail
  /* Info Banner */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    style: "#F0F2FC",
    showBorder: false,
    marginAbove: "STANDARD",
    contents: {
      a!sideBySideLayout(
        spacing: "STANDARD",
        items: {
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextIcon(
                icon: "info-circle",
                color: "#143CCC",
                size: "STANDARD"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: {
                a!richTextItem(
                  text: "A new Case Management System is available. Contact your Administrator with any questions."
                ),
                a!richTextItem(
                  text: " Learn more",
                  color: "ACCENT",
                  link: a!dynamicLink(),
                  linkStyle: "STANDALONE"
                )
              },
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "AUTO"
          )
        },
        alignVertical: "TOP",
        marginAbove: "STANDARD",
        marginBelow: "STANDARD"
      )
    }
  ),
  /* Closed Banner */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    style: "#F2F2F5",
    showBorder: false,
    marginAbove: "STANDARD",
    contents: {
      a!sideBySideLayout(
        spacing: "STANDARD",
        items: {
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextIcon(
                icon: "lock",
                color: "#5C5C66",
                size: "STANDARD"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextItem(
                text: "Case #1123 has been locked. A survey has been sent to the customer."
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "AUTO"
          )
        },
        alignVertical: "TOP",
        marginAbove: "STANDARD",
        marginBelow: "STANDARD"
      )
    }
  ),
  /* Success Banner */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    style: "#EDFCEA",
    showBorder: false,
    marginAbove: "STANDARD",
    contents: {
      a!sideBySideLayout(
        spacing: "STANDARD",
        items: {
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextIcon(
                icon: "check-circle",
                color: "#24990F",
                size: "STANDARD"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextItem(
                text: "Case #1123 has been closed. A survey has been sent to the customer."
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "AUTO"
          )
        },
        alignVertical: "TOP",
        marginAbove: "STANDARD",
        marginBelow: "STANDARD"
      )
    }
  ),
  /* Warning Banner */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    style: "#FFF9DB",
    showBorder: false,
    marginAbove: "STANDARD",
    contents: {
      a!sideBySideLayout(
        spacing: "STANDARD",
        items: {
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextIcon(
                icon: "exclamation-triangle",
                color: "#E5BF00",
                size: "STANDARD"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: {
                a!richTextItem(
                  text: "The following case has been open for more than 30 days:"
                ),
                a!richTextItem(
                  text: " Case #1124",
                  color: "ACCENT",
                  link: a!dynamicLink(),
                  linkStyle: "STANDALONE"
                )
              },
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "AUTO"
          )
        },
        alignVertical: "TOP",
        marginAbove: "STANDARD",
        marginBelow: "STANDARD"
      )
    }
  ),
  /* Error Banner */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    style: "#FFEFEF",
    showBorder: false,
    marginAbove: "STANDARD",
    contents: {
      a!sideBySideLayout(
        spacing: "STANDARD",
        items: {
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextIcon(
                icon: "exclamation-triangle",
                color: "#B22D2D",
                size: "STANDARD"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextItem(
                text: "Case #1125 is missing. Please notify your Administrator."
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "AUTO"
          )
        },
        alignVertical: "TOP",
        marginAbove: "STANDARD",
        marginBelow: "STANDARD"
      )
    }
  )
  ```

## Messages with Actions

Use buttons or links to enable the user to take actions if needed. Actions may include opening a dialog, expanding or collapsing to view more information, or dismissing the banner for example.

If applying a single button, use `style: "OUTLINE"`. For two actions, use `style: "SOLID", color: "ACCENT"` for the prominent action and `style: "LINK"` for the secondary action. Use SMALL size for all buttons. Avoid placing more than two actions in a banner.

If using a link, use the LINK parameter in the a!richTextItem() component. Avoid placing links adjacent to each other to prevent errors due to mistaken clicks. Set the LINKSTYLE to STANDALONE.

Note: When using the ‘X’ action to dismiss the banner, specific ‘Dismiss {insert name of item}’ in the accessibilityText parameter.

```sail
  /* Info Banner with Dismiss Button */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    style: "#F0F2FC",
    showBorder: false,
    marginAbove: "STANDARD",
    contents: {
      a!sideBySideLayout(
        spacing: "STANDARD",
        items: {
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextIcon(
                icon: "info-circle",
                color: "#143CCC",
                size: "STANDARD"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: {
                a!richTextItem(
                  text: "A new Case Management System is available. Contact your Administrator with any questions."
                ),
                a!richTextItem(
                  text: " Learn more",
                  color: "ACCENT",
                  link: a!dynamicLink(),
                  linkStyle: "STANDALONE"
                )
              },
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "AUTO"
          ),
          a!sideBySideItem(
            item: a!buttonArrayLayout(
              align: "END",
              marginAbove: "NONE",
              marginBelow: "NONE",
              buttons: {
                a!buttonWidget(
                  size: "SMALL",
                  style: "SOLID",
                  label: "Dismiss"
                )
              }
            ),
            width: "MINIMIZE"
          )
        },
        alignVertical: "MIDDLE",
        marginAbove: "STANDARD",
        marginBelow: "STANDARD"
      )
    }
  ),
  /* Success Banner with Go to Log Link */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    style: "#EDFCEA",
    showBorder: false,
    marginAbove: "STANDARD",
    contents: {
      a!sideBySideLayout(
        spacing: "STANDARD",
        items: {
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextIcon(
                icon: "check-circle",
                color: "#24990F",
                size: "STANDARD"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextItem(
                text: "Case #1123 has been closed. A survey has been sent to the customer."
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "AUTO"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextItem(
                text: "Go to log",
                color: "#2322f0",
                link: a!dynamicLink(),
                linkStyle: "STANDALONE"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          )
        },
        alignVertical: "MIDDLE",
        marginAbove: "STANDARD",
        marginBelow: "STANDARD"
      )
    }
  ),
  /* Warning Banner with Close Icon */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    style: "#FFF9DB",
    showBorder: false,
    marginAbove: "STANDARD",
    contents: {
      a!sideBySideLayout(
        spacing: "STANDARD",
        items: {
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextIcon(
                icon: "exclamation-triangle",
                color: "#E5BF00",
                size: "STANDARD"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: {
                a!richTextItem(
                  text: "The following case has been open for more than 30 days:"
                ),
                a!richTextItem(
                  text: " Case #1124",
                  color: "ACCENT",
                  link: a!dynamicLink(),
                  linkStyle: "STANDALONE"
                )
              },
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "AUTO"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextIcon(
                icon: "close",
                color: "#000000",
                link: a!dynamicLink(),
                linkStyle: "STANDALONE"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          )
        },
        alignVertical: "MIDDLE",
        marginAbove: "STANDARD",
        marginBelow: "STANDARD"
      )
    }
  ),
  /* Error Banner with Ignore and View Errors Buttons */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    style: "#FFEFEF",
    showBorder: false,
    marginAbove: "STANDARD",
    contents: {
      a!sideBySideLayout(
        spacing: "STANDARD",
        items: {
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextIcon(
                icon: "exclamation-triangle",
                color: "#B22D2D",
                size: "STANDARD"
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "MINIMIZE"
          ),
          a!sideBySideItem(
            item: a!richTextDisplayField(
              labelPosition: "COLLAPSED",
              value: a!richTextItem(
                text: "Case #1125 is missing. Please notify your Administrator."
              ),
              marginAbove: "NONE",
              marginBelow: "NONE"
            ),
            width: "AUTO"
          ),
          a!sideBySideItem(
            item: a!buttonArrayLayout(
              align: "END",
              marginAbove: "NONE",
              marginBelow: "NONE",
              buttons: {
                a!buttonWidget(
                  size: "SMALL",
                  style: "LINK",
                  label: "Ignore"
                ),
                a!buttonWidget(
                  size: "SMALL",
                  style: "SOLID",
                  label: "View Errors"
                )
              }
            ),
            width: "MINIMIZE"
          )
        },
        alignVertical: "MIDDLE",
        marginAbove: "STANDARD",
        marginBelow: "STANDARD"
      )
    }
  )
```

## Persistent Messages
Use this for messages that are always going to be a part of the UI. It is up to the designer’s discretion if the border is needed or not based on the UI.

```sail
  /* Info Banner */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    showBorder: true,
    marginAbove: "STANDARD",
    contents: {
      a!columnsLayout(
        alignVertical: "MIDDLE",
        spacing: "DENSE",
        columns: {
          a!columnLayout(
            width: "EXTRA_NARROW",
            contents: {
              a!cardLayout(
                showBorder: false,
                style: "#F0F2FC",
                padding: "STANDARD",
                shape: "SEMI_ROUNDED",
                contents: {
                  a!richTextDisplayField(
                    labelPosition: "COLLAPSED",
                    align: "CENTER",
                    marginAbove: "EVEN_LESS",
                    marginBelow: "EVEN_LESS",
                    value: a!richTextIcon(icon: "info", color: "#143CCC")
                  )
                }
              )
            }
          ),
          a!columnLayout(
            contents: {
              a!headingField(
                text: "The URL you have entered is not valid",
                size: "EXTRA_SMALL",
                headingTag: "H3",
                marginAbove: "NONE",
                marginBelow: "NONE"
              ),
              a!richTextDisplayField(
                labelPosition: "COLLAPSED",
                marginAbove: "NONE",
                marginBelow: "NONE",
                value: a!richTextItem(
                  text: "Please check the Web Address in the configuration panel"
                )
              )
            }
          )
        }
      )
    }
  ),
  /* Closed Banner */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    showBorder: true,
    marginAbove: "STANDARD",
    contents: {
      a!columnsLayout(
        alignVertical: "MIDDLE",
        spacing: "DENSE",
        columns: {
          a!columnLayout(
            width: "EXTRA_NARROW",
            contents: {
              a!cardLayout(
                showBorder: false,
                style: "#F2F2F5",
                padding: "STANDARD",
                shape: "SEMI_ROUNDED",
                contents: {
                  a!richTextDisplayField(
                    labelPosition: "COLLAPSED",
                    align: "CENTER",
                    marginAbove: "EVEN_LESS",
                    marginBelow: "EVEN_LESS",
                    value: a!richTextIcon(icon: "lock", color: "#5C5C66")
                  )
                }
              )
            }
          ),
          a!columnLayout(
            contents: {
              a!headingField(
                text: "The URL you have entered is not valid",
                size: "EXTRA_SMALL",
                headingTag: "H3",
                marginAbove: "NONE",
                marginBelow: "NONE"
              ),
              a!richTextDisplayField(
                labelPosition: "COLLAPSED",
                marginAbove: "NONE",
                marginBelow: "NONE",
                value: a!richTextItem(
                  text: "Please check the Web Address in the configuration panel"
                )
              )
            }
          )
        }
      )
    }
  ),
  /* Success Banner */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    showBorder: true,
    marginAbove: "STANDARD",
    contents: {
      a!columnsLayout(
        alignVertical: "MIDDLE",
        spacing: "DENSE",
        columns: {
          a!columnLayout(
            width: "EXTRA_NARROW",
            contents: {
              a!cardLayout(
                showBorder: false,
                style: "#EDFCEA",
                padding: "STANDARD",
                shape: "SEMI_ROUNDED",
                contents: {
                  a!richTextDisplayField(
                    labelPosition: "COLLAPSED",
                    align: "CENTER",
                    marginAbove: "EVEN_LESS",
                    marginBelow: "EVEN_LESS",
                    value: a!richTextIcon(icon: "check-circle", color: "#24990F")
                  )
                }
              )
            }
          ),
          a!columnLayout(
            contents: {
              a!headingField(
                text: "The URL you have entered is not valid",
                size: "EXTRA_SMALL",
                headingTag: "H3",
                marginAbove: "NONE",
                marginBelow: "NONE"
              ),
              a!richTextDisplayField(
                labelPosition: "COLLAPSED",
                marginAbove: "NONE",
                marginBelow: "NONE",
                value: a!richTextItem(
                  text: "Please check the Web Address in the configuration panel"
                )
              )
            }
          )
        }
      )
    }
  ),
  /* Warning Banner */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    showBorder: true,
    marginAbove: "STANDARD",
    contents: {
      a!columnsLayout(
        alignVertical: "MIDDLE",
        spacing: "DENSE",
        columns: {
          a!columnLayout(
            width: "EXTRA_NARROW",
            contents: {
              a!cardLayout(
                showBorder: false,
                style: "#FFF9DB",
                padding: "STANDARD",
                shape: "SEMI_ROUNDED",
                contents: {
                  a!richTextDisplayField(
                    labelPosition: "COLLAPSED",
                    align: "CENTER",
                    marginAbove: "EVEN_LESS",
                    marginBelow: "EVEN_LESS",
                    value: a!richTextIcon(
                      icon: "exclamation-triangle",
                      color: "#E5BF00"
                    )
                  )
                }
              )
            }
          ),
          a!columnLayout(
            contents: {
              a!headingField(
                text: "The URL you have entered is not valid",
                size: "EXTRA_SMALL",
                headingTag: "H3",
                marginAbove: "NONE",
                marginBelow: "NONE"
              ),
              a!richTextDisplayField(
                labelPosition: "COLLAPSED",
                marginAbove: "NONE",
                marginBelow: "NONE",
                value: a!richTextItem(
                  text: "Please check the Web Address in the configuration panel"
                )
              )
            }
          )
        }
      )
    }
  ),
  /* Error Banner */
  a!cardLayout(
    shape: "SEMI_ROUNDED",
    showBorder: true,
    marginAbove: "STANDARD",
    contents: {
      a!columnsLayout(
        alignVertical: "MIDDLE",
        spacing: "DENSE",
        columns: {
          a!columnLayout(
            width: "EXTRA_NARROW",
            contents: {
              a!cardLayout(
                showBorder: false,
                style: "#FFEFEF",
                padding: "STANDARD",
                shape: "SEMI_ROUNDED",
                contents: {
                  a!richTextDisplayField(
                    labelPosition: "COLLAPSED",
                    align: "CENTER",
                    marginAbove: "EVEN_LESS",
                    marginBelow: "EVEN_LESS",
                    value: a!richTextIcon(
                      icon: "exclamation-triangle",
                      color: "#B22D2D"
                    )
                  )
                }
              )
            }
          ),
          a!columnLayout(
            contents: {
              a!headingField(
                text: "The URL you have entered is not valid",
                size: "EXTRA_SMALL",
                headingTag: "H3",
                marginAbove: "NONE",
                marginBelow: "NONE"
              ),
              a!richTextDisplayField(
                labelPosition: "COLLAPSED",
                marginAbove: "NONE",
                marginBelow: "NONE",
                value: a!richTextItem(
                  text: "Please check the Web Address in the configuration panel"
                )
              )
            }
          )
        }
      )
    }
  )
```