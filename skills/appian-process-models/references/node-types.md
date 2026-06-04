# Process Model Node Types

Complete reference for all supported node types, their activity class schema IDs, configuration shapes, and examples.

## Tooled Node Types

These node types are fully supported when creating or updating process models.

---

### Start Event

- **Schema ID**: `core.0`
- **Description**: Entry point for the process model. Exactly one per process model. No configuration needed.
- **Configuration**: None

**Example**:
```json
{
  "id": 1,
  "type": "core.0",
  "name": "Start",
  "coordinates": [50, 200],
  "connections": [2]
}
```

---

### End Event

- **Schema ID**: `core.1`
- **Description**: Termination point for the process model. Connections must be an empty list. Multiple End Events are allowed (e.g., one per branch).
- **Configuration**: None

**Example**:
```json
{
  "id": 5,
  "type": "core.1",
  "name": "End",
  "coordinates": [850, 200],
  "connections": []
}
```

---

### XOR (Exclusive Gateway)

- **Schema ID**: `core.4`
- **Description**: Conditional branching — evaluates conditions in order and routes to the first matching path. Exactly one path executes. A default path handles the case where no condition matches.

**Configuration**:

| Parameter | Type | Description |
|---|---|---|
| `conditions` | list | List of condition objects, each with `condition` (expression string) and `path` (target node ID) |
| `defaultPath` | int | Node ID for the default path when no condition matches |

- Each `condition` is an Appian expression that evaluates to Boolean
- `path` is the node ID to route to when the condition is true
- Conditions are evaluated in order; the first `true` condition wins
- `defaultPath` is taken when all conditions evaluate to `false`
- Any `recordType!` references in condition expressions must use the UUID-qualified format — see the Appian Expressions skill

**Example** — Cancel check after a form:
```json
{
  "id": 3,
  "type": "core.4",
  "name": "Is Cancelled?",
  "coordinates": [450, 200],
  "connections": [4, 5],
  "configurations": {
    "conditions": [
      {
        "condition": "=pv!cancel",
        "path": 5
      }
    ],
    "defaultPath": 4
  }
}
```

**Example** — Status-based routing:
```json
{
  "id": 3,
  "type": "core.4",
  "name": "Check Priority",
  "coordinates": [450, 200],
  "connections": [4, 5, 6],
  "configurations": {
    "conditions": [
      {
        "condition": "=pv!priority = \"HIGH\"",
        "path": 4
      },
      {
        "condition": "=pv!priority = \"MEDIUM\"",
        "path": 5
      }
    ],
    "defaultPath": 6
  }
}
```

---

### Script Task

- **Schema ID**: `internal.16`
- **Description**: Executes expressions and assigns results to process variables. Use for data transformations, calculations, and variable assignments without user interaction.

**Configuration**:

| Parameter | Type | Description |
|---|---|---|
| `expressionBody` | string | Appian expression to execute (must start with `=`) |
| `outputVariables` | list | List of output variable assignments, each with `name` (process variable name) and `expression` (value expression) |

- `expressionBody` runs first, then `outputVariables` are assigned
- Use `outputVariables` to save computed values into process variables
- Expressions can reference process variables via `pv!variableName`
- All `recordType!` references in expressions must use the UUID-qualified format: `'recordType!{rtUuid}Name.fields.{fieldUuid}fieldName'` (with single quotes wrapping the entire reference) — see the Appian Expressions skill for details

**Example** — Set default values:
```json
{
  "id": 2,
  "type": "internal.16",
  "name": "Set Defaults",
  "coordinates": [250, 200],
  "connections": [3],
  "configurations": {
    "outputVariables": [
      {
        "name": "status",
        "expression": "=\"Open\""
      },
      {
        "name": "createdDate",
        "expression": "=today()"
      }
    ]
  }
}
```

**Example** — Transform data:
```json
{
  "id": 4,
  "type": "internal.16",
  "name": "Calculate Risk Score",
  "coordinates": [650, 200],
  "connections": [5],
  "configurations": {
    "expressionBody": "=rule!ACME_CalculateRiskScore(caseRecord: pv!record)",
    "outputVariables": [
      {
        "name": "riskScore",
        "expression": "=rule!ACME_CalculateRiskScore(caseRecord: pv!record)"
      }
    ]
  }
}
```

---

### User Input Task

- **Schema ID**: `internal.17`
- **Description**: Presents an interface form to a user and pauses the process until the user submits or cancels. The interface must exist before the process model references it.

**Configuration**:

| Parameter | Type | Description |
|---|---|---|
| `interfaceUuid` | string | UUID of the interface to display |
| `formInputs` | list | List of input mappings, each with `name` (interface input name) and `expression` (process variable or value) |

- `name` must exactly match an input defined on the interface
- `expression` maps a process variable or literal to the interface input (e.g., `pv!record`, `false`)
- After the user submits, values saved by the interface flow back into the mapped process variables

**Example** — Edit form:
```json
{
  "id": 2,
  "type": "internal.17",
  "name": "Edit Case",
  "coordinates": [250, 200],
  "connections": [3],
  "configurations": {
    "interfaceUuid": "<interface-uuid>",
    "formInputs": [
      { "name": "record", "expression": "pv!record" },
      { "name": "isUpdate", "expression": "true" },
      { "name": "cancel", "expression": "pv!cancel" }
    ]
  }
}
```

---

### Write Records

- **Schema ID**: `internal3.write_records_to_source_23r3`
- **Description**: Writes a record to the data source (creates or updates depending on whether the primary key is populated). Requires a record type to be specified.

**Configuration**:

| Parameter | Type | Description |
|---|---|---|
| `record` | string | Process variable expression containing the record data (e.g., `pv!record`) |
| `recordTypeUuid` | string | UUID of the record type to write to |

- If the record's primary key field is null/empty, a new record is created
- If the primary key is populated, the existing record is updated
- The record type must exist and have a database source configuration

**Example**:
```json
{
  "id": 4,
  "type": "internal3.write_records_to_source_23r3",
  "name": "Write Case Record",
  "coordinates": [650, 200],
  "connections": [5],
  "configurations": {
    "record": "pv!record",
    "recordTypeUuid": "<record-type-uuid>"
  }
}
```

---

### Send E-Mail

- **Schema ID**: `internal3.sendemail3`
- **Description**: Sends an email to specified recipients. Supports expressions for dynamic content.

**Configuration**:

| Parameter | Type | Description |
|---|---|---|
| `recipient` | string | Expression resolving to the email recipient(s) (e.g., a process variable holding a user or email address) |
| `message` | string | Email body content — can include expressions for dynamic content |

**Example**:
```json
{
  "id": 3,
  "type": "internal3.sendemail3",
  "name": "Send Confirmation Email",
  "coordinates": [450, 200],
  "connections": [4],
  "configurations": {
    "recipient": "pv!assignedUser",
    "message": "Your case has been created successfully."
  }
}
```

---

## Untooled Node Types

These node types can be included in process model definitions but represent capabilities without dedicated MCP tools. The underlying AI skills, integrations, or database queries require manual configuration in the Appian environment.

---

### Classify Documents

- **Schema ID**: `internal3.rs2_ai_skill_document_classification3`
- **Description**: AI skill node that classifies documents into categories. Requires an AI skill to be configured in the Appian environment.
- **Configuration**: AI skill-specific — configure in the Appian Designer after creating the process model.
- **Manual step**: Create and configure the document classification AI skill in Appian before this node can execute.

**Example** (placeholder node):
```json
{
  "id": 3,
  "type": "internal3.rs2_ai_skill_document_classification3",
  "name": "Classify Document",
  "coordinates": [450, 200],
  "connections": [4]
}
```

---

### Classify Emails

- **Schema ID**: `internal3.rs2_ai_skill_email_classification2`
- **Description**: AI skill node that classifies emails by intent or category. Requires an AI skill to be configured in the Appian environment.
- **Configuration**: AI skill-specific — configure in the Appian Designer after creating the process model.
- **Manual step**: Create and configure the email classification AI skill in Appian before this node can execute.

**Example** (placeholder node):
```json
{
  "id": 3,
  "type": "internal3.rs2_ai_skill_email_classification2",
  "name": "Classify Email",
  "coordinates": [450, 200],
  "connections": [4]
}
```

---

### Extract from Document

- **Schema ID**: `internal3.rs2_ai_skill_document_extraction23r4`
- **Description**: AI skill node that extracts structured data from documents. Requires an AI skill to be configured in the Appian environment.
- **Configuration**: AI skill-specific — configure in the Appian Designer after creating the process model.
- **Manual step**: Create and configure the document extraction AI skill in Appian before this node can execute.

**Example** (placeholder node):
```json
{
  "id": 3,
  "type": "internal3.rs2_ai_skill_document_extraction23r4",
  "name": "Extract Document Data",
  "coordinates": [450, 200],
  "connections": [4]
}
```

---

### Query Database

- **Schema ID**: `internal.database601`
- **Description**: Executes a direct database query. Use when you need to run SQL or stored procedures outside of Appian's record type data layer.
- **Configuration**: Database-specific — configure the query, data source, and result mapping in the Appian Designer after creating the process model.
- **Manual step**: Configure the database query details in Appian Designer.

**Example** (placeholder node):
```json
{
  "id": 3,
  "type": "internal.database601",
  "name": "Query Legacy Database",
  "coordinates": [450, 200],
  "connections": [4]
}
```

---

### Call Integration

- **Schema ID**: `internal3.integration`
- **Description**: Invokes an integration (connected system) to call external APIs or services. Requires the integration and connected system to be configured in the Appian environment.
- **Configuration**: Integration-specific — configure the integration object, connected system, and input/output mappings in the Appian Designer after creating the process model.
- **Manual step**: Create the connected system and integration object in Appian before this node can execute.

**Example** (placeholder node):
```json
{
  "id": 3,
  "type": "internal3.integration",
  "name": "Call External API",
  "coordinates": [450, 200],
  "connections": [4]
}
```

---

## Complete Process Model Examples

### Record Create Process (Form → Write)

A standard process for creating a new record via a form:

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "core.0",
      "name": "Start",
      "coordinates": [50, 200],
      "connections": [2]
    },
    {
      "id": 2,
      "type": "internal.17",
      "name": "Enter Case Details",
      "coordinates": [250, 200],
      "connections": [3],
      "configurations": {
        "interfaceUuid": "<case-form-uuid>",
        "formInputs": [
          { "name": "record", "expression": "pv!record" },
          { "name": "isUpdate", "expression": "false" },
          { "name": "cancel", "expression": "pv!cancel" }
        ]
      }
    },
    {
      "id": 3,
      "type": "core.4",
      "name": "Is Cancelled?",
      "coordinates": [450, 200],
      "connections": [4, 5],
      "configurations": {
        "conditions": [
          { "condition": "=pv!cancel", "path": 5 }
        ],
        "defaultPath": 4
      }
    },
    {
      "id": 4,
      "type": "internal3.write_records_to_source_23r3",
      "name": "Write Case Record",
      "coordinates": [650, 200],
      "connections": [5],
      "configurations": {
        "record": "pv!record",
        "recordTypeUuid": "<case-record-type-uuid>"
      }
    },
    {
      "id": 5,
      "type": "core.1",
      "name": "End",
      "coordinates": [850, 200],
      "connections": []
    }
  ],
  "variables": [
    {
      "name": "record",
      "type": "INTEGER",
      "isParameter": true,
      "isRequired": false,
      "recordTypeUuid": "<case-record-type-uuid>"
    },
    {
      "name": "cancel",
      "type": "BOOLEAN"
    }
  ]
}
```

### Record Action Process (Form → Conditional Write)

A record action process that operates on an existing record:

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "core.0",
      "name": "Start",
      "coordinates": [50, 200],
      "connections": [2]
    },
    {
      "id": 2,
      "type": "internal.17",
      "name": "Reassign Case",
      "coordinates": [250, 200],
      "connections": [3],
      "configurations": {
        "interfaceUuid": "<reassign-form-uuid>",
        "formInputs": [
          { "name": "record", "expression": "pv!record" },
          { "name": "cancel", "expression": "pv!cancel" }
        ]
      }
    },
    {
      "id": 3,
      "type": "core.4",
      "name": "Is Cancelled?",
      "coordinates": [450, 200],
      "connections": [4, 5],
      "configurations": {
        "conditions": [
          { "condition": "=pv!cancel", "path": 5 }
        ],
        "defaultPath": 4
      }
    },
    {
      "id": 4,
      "type": "internal3.write_records_to_source_23r3",
      "name": "Write Updated Record",
      "coordinates": [650, 200],
      "connections": [5],
      "configurations": {
        "record": "pv!record",
        "recordTypeUuid": "<case-record-type-uuid>"
      }
    },
    {
      "id": 5,
      "type": "core.1",
      "name": "End",
      "coordinates": [850, 200],
      "connections": []
    }
  ],
  "variables": [
    {
      "name": "record",
      "type": "INTEGER",
      "isParameter": true,
      "isRequired": true,
      "recordTypeUuid": "<case-record-type-uuid>"
    },
    {
      "name": "cancel",
      "type": "BOOLEAN"
    }
  ]
}
```

### Script-Only Process (No User Interaction)

A utility process that runs logic without user input:

```json
{
  "nodes": [
    {
      "id": 1,
      "type": "core.0",
      "name": "Start",
      "coordinates": [50, 200],
      "connections": [2]
    },
    {
      "id": 2,
      "type": "internal.16",
      "name": "Calculate Values",
      "coordinates": [250, 200],
      "connections": [3],
      "configurations": {
        "outputVariables": [
          {
            "name": "riskScore",
            "expression": "=rule!ACME_CalculateRiskScore(caseRecord: pv!record)"
          }
        ]
      }
    },
    {
      "id": 3,
      "type": "internal3.sendemail3",
      "name": "Notify Assignee",
      "coordinates": [450, 200],
      "connections": [4],
      "configurations": {
        "recipient": "pv!assignedUser",
        "message": "A new case has been assigned to you with risk score: " & pv!riskScore"
      }
    },
    {
      "id": 4,
      "type": "core.1",
      "name": "End",
      "coordinates": [650, 200],
      "connections": []
    }
  ],
  "variables": [
    {
      "name": "record",
      "type": "INTEGER",
      "isParameter": true,
      "isRequired": true,
      "recordTypeUuid": "<case-record-type-uuid>"
    },
    {
      "name": "assignedUser",
      "type": "USER",
      "isParameter": true,
      "isRequired": true
    },
    {
      "name": "riskScore",
      "type": "NUMBER"
    }
  ]
}
```
