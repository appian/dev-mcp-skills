# Write Records Patterns

**Purpose:** Patterns for saving data from interfaces - inline writes vs process-based updates.

**When to use:** Building editable forms with Save buttons that need to persist changes to record types.

---

## Decision Tree: Inline vs Process

```
Need to save form data to a record?
├─ Does an "Update" or "Modify" process model exist for this record type?
│  │
│  ├─ NO → Use a!writeRecords() inline (Section 1)
│  │  - Simple CRUD forms
│  │  - No workflow/approval needed
│  │  - Immediate save on button click
│  │  - Good for: status updates, field edits, quick changes
│  │
│  └─ YES → Use a!startProcess() (Section 2)
│     - Complex updates with workflow
│     - Needs approval, notifications, or multi-step logic
│     - Process handles Write Records node
│     - Good for: approval flows, multi-record updates, business rules
│
└─ Creating NEW records (not updating)?
   - Same logic applies
   - Look for "Create" process model
   - If none exists, use a!writeRecords() inline
```

**How to check if process exists:**
```sail
/* Use listProcessModels MCP tool filtered by record type name */
/* Look for process names containing: "Update", "Modify", "Edit", "Save" */
/* Example: "Update Order", "Modify Customer", "Edit Ticket" */
```

---

## Section 1: Inline Updates with a!writeRecords()

**Use when:**
- No update/modify process model exists
- Simple field updates (no complex workflow)
- Immediate save without approval

### Basic Pattern (Single Record Update)

```sail
a!localVariables(
  /* Query the record to edit */
  local!order: a!queryRecordByIdentifier(
    recordType: 'recordType!Order',
    identifier: ri!orderId,
    fields: {
      'recordType!Order.fields.id',
      'recordType!Order.fields.customerName',
      'recordType!Order.fields.totalAmount',
      'recordType!Order.fields.statusId',
      'recordType!Order.fields.notes'
    }
  ),
  
  /* Local variables to track form state */
  local!customerName: a!defaultValue(local!order['recordType!Order.fields.customerName'], ""),
  local!totalAmount: a!defaultValue(local!order['recordType!Order.fields.totalAmount'], 0),
  local!statusId: a!defaultValue(local!order['recordType!Order.fields.statusId'], null),
  local!notes: a!defaultValue(local!order['recordType!Order.fields.notes'], ""),
  
  /* Save success/error tracking */
  local!saveSuccess: false,
  local!saveError: null,
  
  a!formLayout(
    label: "Edit Order",
    contents: {
      /* Success message banner */
      if(
        local!saveSuccess,
        a!sectionLayout(
          contents: {
            a!richTextDisplayField(
              value: a!richTextItem(
                text: "Order updated successfully!",
                color: "POSITIVE",
                style: "STRONG"
              ),
              labelPosition: "COLLAPSED"
            )
          },
          showBorder: false
        ),
        {}
      ),
      
      /* Error message banner */
      if(
        a!isNotNullOrEmpty(local!saveError),
        a!sectionLayout(
          contents: {
            a!richTextDisplayField(
              value: a!richTextItem(
                text: "Error saving order: " & local!saveError,
                color: "NEGATIVE",
                style: "STRONG"
              ),
              labelPosition: "COLLAPSED"
            )
          },
          showBorder: false
        ),
        {}
      ),
      
      /* Form fields */
      a!sectionLayout(
        label: "Order Details",
        contents: {
          a!textField(
            label: "Customer Name",
            value: local!customerName,
            saveInto: local!customerName,
            required: true
          ),
          a!floatingPointField(
            label: "Total Amount",
            value: local!totalAmount,
            saveInto: local!totalAmount,
            required: true
          ),
          a!dropdownField(
            label: "Status",
            choiceLabels: {"Pending", "Confirmed", "Shipped", "Delivered"},
            choiceValues: {1, 2, 3, 4},
            value: local!statusId,
            saveInto: local!statusId,
            required: true
          ),
          a!paragraphField(
            label: "Notes",
            value: local!notes,
            saveInto: local!notes
          )
        }
      )
    },
    buttons: a!buttonLayout(
      primaryButtons: {
        a!buttonWidget(
          label: "Save",
          saveInto: {
            a!writeRecords(
              records: {
                'recordType!Order'(
                  'recordType!Order.fields.id': ri!orderId,
                  'recordType!Order.fields.customerName': local!customerName,
                  'recordType!Order.fields.totalAmount': local!totalAmount,
                  'recordType!Order.fields.statusId': local!statusId,
                  'recordType!Order.fields.notes': local!notes
                )
              },
              onSuccess: {
                a!save(local!saveSuccess, true),
                a!save(local!saveError, null)
              },
              onError: {
                a!save(local!saveSuccess, false),
                a!save(local!saveError, fv!error)
              }
            )
          },
          submit: true,
          validate: true,
          style: "SOLID"
        )
      },
      secondaryButtons: {
        a!buttonWidget(
          label: "Cancel",
          value: true,
          saveInto: {},
          style: "OUTLINE"
        )
      }
    )
  )
)
```

### Key Components Explained

**1. Record Constructor Syntax**
```sail
'recordType!Order'(
  'recordType!Order.fields.id': ri!orderId,        /* Primary key - REQUIRED for updates */
  'recordType!Order.fields.customerName': local!customerName,
  'recordType!Order.fields.totalAmount': local!totalAmount
)
```
- Uses record type as a constructor (like a function call)
- Primary key field (`id`) is **REQUIRED** for updates
- Only include fields you want to update (omitted fields stay unchanged)
- Field references use full qualified names: `'recordType!Name.fields.fieldName'`

**2. onSuccess / onError Handlers**
```sail
onSuccess: {
  a!save(local!saveSuccess, true),
  a!save(local!saveError, null)
}
```
- `onSuccess`: Executed when write succeeds
- `onError`: Executed when write fails (use `fv!error` to get error message)
- Update local variables to show feedback banners

**3. Submit and Validate**
```sail
submit: true,    /* Form submission (triggers onSuccess/onError) */
validate: true   /* Run form validations before save */
```

---

### Multi-Record Updates

**Updating multiple records in one save:**

```sail
a!buttonWidget(
  label: "Save All",
  saveInto: {
    a!writeRecords(
      records: {
        /* Update Order */
        'recordType!Order'(
          'recordType!Order.fields.id': ri!orderId,
          'recordType!Order.fields.totalAmount': local!orderTotal,
          'recordType!Order.fields.statusId': local!statusId
        ),
        /* Update Customer (related record) */
        'recordType!Customer'(
          'recordType!Customer.fields.id': local!customerId,
          'recordType!Customer.fields.loyaltyPoints': local!updatedPoints
        )
      },
      onSuccess: {
        a!save(local!saveSuccess, true)
      },
      onError: {
        a!save(local!saveError, fv!error)
      }
    )
  },
  submit: true
)
```

**When to use:**
- Updating primary record + related record together
- Batch updates (e.g., bulk status changes)
- Transactional updates (all succeed or all fail)

---

### Creating New Records

**Pattern for "Create" forms:**

```sail
a!buttonWidget(
  label: "Create Order",
  saveInto: {
    a!writeRecords(
      records: {
        'recordType!Order'(
          /* NO id field - Appian auto-generates primary key */
          'recordType!Order.fields.customerName': local!customerName,
          'recordType!Order.fields.totalAmount': local!totalAmount,
          'recordType!Order.fields.statusId': 1,  /* Default to "Pending" */
          'recordType!Order.fields.createdBy': loggedInUser(),
          'recordType!Order.fields.createdAt': now()
        )
      },
      onSuccess: {
        a!save(local!newOrderId, fv!newRecords[1]['recordType!Order.fields.id']),
        a!save(local!saveSuccess, true)
      },
      onError: {
        a!save(local!saveError, fv!error)
      }
    )
  },
  submit: true
)
```

**Key differences from updates:**
- **Omit primary key field** (`id`) - Appian auto-generates it
- Use `fv!newRecords` in `onSuccess` to get the created record(s) with new IDs
- Explicitly set audit fields: `createdBy`, `createdAt`

---

### Limitations of a!writeRecords()

**What a!writeRecords() CANNOT do:**
- ❌ Write record events (use Write Records node in process model)
- ❌ Execute complex business logic (validations beyond required fields)
- ❌ Send notifications
- ❌ Start approval workflows
- ❌ Update multiple related records with conditional logic
- ❌ Call integrations (REST/SOAP)

**When you hit these limitations → Use a!startProcess() instead (Section 2)**

---

## Section 2: Process-Based Updates with a!startProcess()

**Use when:**
- Update/Modify process model exists
- Need workflow (approval, notifications, business rules)
- Complex multi-step logic
- Integration calls required

### Basic Pattern

```sail
a!localVariables(
  /* Query the record to edit */
  local!product: a!queryRecordByIdentifier(
    recordType: 'recordType!Product',
    identifier: ri!productId,
    fields: {
      'recordType!Product.fields.id',
      'recordType!Product.fields.name',
      'recordType!Product.fields.price',
      'recordType!Product.fields.categoryId'
    }
  ),
  
  /* Local variables to track form state */
  local!name: a!defaultValue(local!product['recordType!Product.fields.name'], ""),
  local!price: a!defaultValue(local!product['recordType!Product.fields.price'], 0),
  local!categoryId: a!defaultValue(local!product['recordType!Product.fields.categoryId'], null),
  
  /* Process start tracking */
  local!processStarted: false,
  local!processError: null,
  
  a!formLayout(
    label: "Edit Product",
    contents: {
      /* Success message */
      if(
        local!processStarted,
        a!sectionLayout(
          contents: {
            a!richTextDisplayField(
              value: a!richTextItem(
                text: "Update submitted successfully! Changes will be processed.",
                color: "POSITIVE",
                style: "STRONG"
              ),
              labelPosition: "COLLAPSED"
            )
          },
          showBorder: false
        ),
        {}
      ),
      
      /* Error message */
      if(
        a!isNotNullOrEmpty(local!processError),
        a!sectionLayout(
          contents: {
            a!richTextDisplayField(
              value: a!richTextItem(
                text: "Error starting update: " & local!processError,
                color: "NEGATIVE",
                style: "STRONG"
              ),
              labelPosition: "COLLAPSED"
            )
          },
          showBorder: false
        ),
        {}
      ),
      
      /* Form fields */
      a!sectionLayout(
        label: "Product Details",
        contents: {
          a!textField(
            label: "Product Name",
            value: local!name,
            saveInto: local!name,
            required: true
          ),
          a!floatingPointField(
            label: "Price",
            value: local!price,
            saveInto: local!price,
            required: true
          ),
          a!dropdownField(
            label: "Category",
            choiceLabels: {"Electronics", "Clothing", "Food", "Books"},
            choiceValues: {1, 2, 3, 4},
            value: local!categoryId,
            saveInto: local!categoryId,
            required: true
          )
        }
      )
    },
    buttons: a!buttonLayout(
      primaryButtons: {
        a!buttonWidget(
          label: "Save",
          saveInto: {
            a!startProcess(
              processModel: cons!UPDATE_PRODUCT_PROCESS,  /* Constant storing process model UUID */
              processParameters: {
                productId: ri!productId,
                name: local!name,
                price: local!price,
                categoryId: local!categoryId
              },
              onSuccess: {
                a!save(local!processStarted, true),
                a!save(local!processError, null)
              },
              onError: {
                a!save(local!processStarted, false),
                a!save(local!processError, fv!error)
              }
            )
          },
          submit: true,
          validate: true,
          style: "SOLID"
        )
      },
      secondaryButtons: {
        a!buttonWidget(
          label: "Cancel",
          value: true,
          saveInto: {},
          style: "OUTLINE"
        )
      }
    )
  )
)
```

### Key Components Explained

**1. Process Model Reference**
```sail
processModel: cons!UPDATE_PRODUCT_PROCESS
```
- Reference the process model UUID via a constant
- Constant naming: `UPDATE_<RECORDTYPE>_PROCESS` or `MODIFY_<RECORDTYPE>_PROCESS`
- Must be an "Update" or "Modify" process (not "Create" with start form)

**2. Process Parameters**
```sail
processParameters: {
  productId: ri!productId,
  name: local!name,
  price: local!price
}
```
- Pass form data to process as parameters
- Parameter names must match process model variable names
- Process model handles Write Records node logic

**3. Finding the Right Process**

**Steps to identify update process:**
1. Use `listProcessModels` MCP tool filtered by application
2. Look for process names containing: "Update", "Modify", "Edit", "Save"
3. Check process model has appropriate input parameters (record ID + fields to update)
4. Verify it's NOT a "Create" process with a start form (those are for new records)

**Example search:**
```
listProcessModels(appUuid: "<app-uuid>", query: "Update")
→ Results: "Update Order", "Update Customer", "Modify Product"
```

---

### When Update Process Doesn't Exist

**Fallback decision:**
```
Found an Update process?
├─ NO → Use a!writeRecords() inline (Section 1)
│  - Create the interface with inline save
│  - Consider creating update process later if complexity grows
│
└─ YES → Use a!startProcess() (this section)
   - Reference the process in saveInto
   - Pass record data as parameters
```

---

## Section 3: Variant Examples

### Example: Customer Record Update

```sail
/* Option 1: Inline with a!writeRecords() */
a!writeRecords(
  records: {
    'recordType!Customer'(
      'recordType!Customer.fields.id': ri!customerId,
      'recordType!Customer.fields.email': local!email,
      'recordType!Customer.fields.phone': local!phone,
      'recordType!Customer.fields.address': local!address
    )
  },
  onSuccess: { a!save(local!saveSuccess, true) },
  onError: { a!save(local!saveError, fv!error) }
)

/* Option 2: Process-based with a!startProcess() */
a!startProcess(
  processModel: cons!UPDATE_CUSTOMER_PROCESS,
  processParameters: {
    customerId: ri!customerId,
    email: local!email,
    phone: local!phone,
    address: local!address
  },
  onSuccess: { a!save(local!processStarted, true) },
  onError: { a!save(local!processError, fv!error) }
)
```

### Example: Ticket Record Update

```sail
/* Option 1: Inline with a!writeRecords() */
a!writeRecords(
  records: {
    'recordType!Ticket'(
      'recordType!Ticket.fields.id': ri!ticketId,
      'recordType!Ticket.fields.priorityId': local!priorityId,
      'recordType!Ticket.fields.statusId': local!statusId,
      'recordType!Ticket.fields.assignedTo': local!assignedTo,
      'recordType!Ticket.fields.notes': local!notes
    )
  },
  onSuccess: { a!save(local!saveSuccess, true) },
  onError: { a!save(local!saveError, fv!error) }
)

/* Option 2: Process-based with a!startProcess() */
a!startProcess(
  processModel: cons!UPDATE_TICKET_PROCESS,
  processParameters: {
    ticketId: ri!ticketId,
    priorityId: local!priorityId,
    statusId: local!statusId,
    assignedTo: local!assignedTo,
    notes: local!notes
  },
  onSuccess: { a!save(local!processStarted, true) },
  onError: { a!save(local!processError, fv!error) }
)
```

---

## Section 4: Common Patterns and Anti-Patterns

### ✅ Correct Patterns

**1. Always include primary key for updates:**
```sail
'recordType!Order'(
  'recordType!Order.fields.id': ri!orderId,  /* REQUIRED */
  'recordType!Order.fields.status': local!status
)
```

**2. Use a!defaultValue() for null safety:**
```sail
local!customerName: a!defaultValue(
  local!order['recordType!Order.fields.customerName'],
  ""  /* Default to empty string if null */
)
```

**3. Clear success/error feedback:**
```sail
if(local!saveSuccess, "Saved!", {}),
if(a!isNotNullOrEmpty(local!saveError), "Error: " & local!saveError, {})
```

**4. Generic record type naming:**
```sail
/* ✅ CORRECT - Generic name, no UUIDs in SAIL code */
'recordType!Order'(
  'recordType!Order.fields.id': ri!orderId,
  'recordType!Order.fields.total': local!total
)

/* Note: MCP tools (createInterface) require UUIDs in JSON,
   but SAIL expressions use generic names. The system
   resolves generic names to UUIDs at runtime. */
```

---

### ❌ Anti-Patterns

**1. Missing primary key in update:**
```sail
/* ❌ WRONG - No id field, Appian doesn't know which record to update */
'recordType!Order'(
  'recordType!Order.fields.customerName': local!name
)

/* ✅ CORRECT */
'recordType!Order'(
  'recordType!Order.fields.id': ri!orderId,  /* Primary key REQUIRED */
  'recordType!Order.fields.customerName': local!name
)
```

**2. Submit without saveInto:**
```sail
/* ❌ WRONG - submit: true but no saveInto = does nothing */
a!buttonWidget(
  label: "Save",
  submit: true,
  validate: true
)

/* ✅ CORRECT */
a!buttonWidget(
  label: "Save",
  saveInto: {
    a!writeRecords(...)
  },
  submit: true
)
```

**3. No error handling:**
```sail
/* ❌ WRONG - No onError, user won't see failures */
a!writeRecords(
  records: {...},
  onSuccess: { a!save(local!saveSuccess, true) }
)

/* ✅ CORRECT */
a!writeRecords(
  records: {...},
  onSuccess: { a!save(local!saveSuccess, true) },
  onError: { a!save(local!saveError, fv!error) }  /* Always handle errors */
)
```

**4. Using UUID placeholders in SAIL code:**
```sail
/* ❌ WRONG - Don't use UUID placeholders in SAIL expressions */
'recordType!{uuid}Order'(
  'recordType!{uuid}Order.fields.{uuid}id': ri!orderId
)

/* ✅ CORRECT - Use generic names only */
'recordType!Order'(
  'recordType!Order.fields.id': ri!orderId
)

/* UUID placeholders are ONLY for documentation examples
   showing where UUIDs go in MCP tool JSON payloads. */
```

**5. Including id field when creating:**
```sail
/* ❌ WRONG - Don't specify id when creating new records */
'recordType!Order'(
  'recordType!Order.fields.id': 0,  /* Appian auto-generates this */
  'recordType!Order.fields.customerName': local!name
)

/* ✅ CORRECT - Omit id for creates */
'recordType!Order'(
  'recordType!Order.fields.customerName': local!name
)
```

---

## Section 5: Testing Checklist

Before calling MCP tools to create interface with save functionality:

- [ ] **Decision made:** Inline (a!writeRecords) vs Process (a!startProcess)?
- [ ] **If process-based:** Process model UUID identified and stored in constant?
- [ ] **Primary key handling:** Included for updates, omitted for creates?
- [ ] **Local variables:** Form state tracked in local! variables?
- [ ] **Success tracking:** local!saveSuccess variable exists?
- [ ] **Error tracking:** local!saveError variable exists?
- [ ] **onSuccess handler:** Updates local!saveSuccess correctly?
- [ ] **onError handler:** Captures fv!error into local!saveError?
- [ ] **Feedback banners:** Success and error messages displayed to user?
- [ ] **Button configuration:** submit: true AND validate: true?
- [ ] **Null safety:** All query results wrapped in a!defaultValue()?

---

## Section 6: Integration with Loading Strategy

**When to load this file:**

From SKILL.md Step 2 (Load Primary Domain Reference):
```
Building interfaces:
├─ Read-only interface (no Save button)? 
│  └─ Load: interfaces.md + sail.md
│
└─ Editable form (has Save button)?
   └─ Load: interfaces.md + sail.md + write-records-patterns.md
      - Decide: inline vs process
      - Implement save button with appropriate pattern
```

**This file is MANDATORY for any interface with a Save button.**

---

## Section 7: Quick Reference

### a!writeRecords() Signature

```sail
a!writeRecords(
  records: {<record constructors>},
  onSuccess: {<actions>},
  onError: {<actions>}
)
```

### a!startProcess() Signature

```sail
a!startProcess(
  processModel: <constant or UUID>,
  processParameters: {<key: value map>},
  onSuccess: {<actions>},
  onError: {<actions>}
)
```

### Record Constructor Pattern

```sail
'recordType!<Name>'(
  'recordType!<Name>.fields.<fieldName>': <value>,
  'recordType!<Name>.fields.<fieldName>': <value>
)
```

---

## Notes

- Replace `Order`, `Customer`, `Product`, `Ticket` with your actual record type name
- Replace field names (`customerName`, `totalAmount`, etc.) with your actual field names
- Use constants for process model UUIDs (e.g., `cons!UPDATE_ORDER_PROCESS`)
- Always test save functionality in Appian before deploying to production
