---
name: "appian-process-models"
description: "Create and modify Appian process models including nodes, variables, connections, and start forms. Covers node types (Script Task, User Input Task, XOR, Write Records, Send E-Mail), process variables, flow connections, and process model folders. Use when building workflows, automating business processes, or connecting interfaces to backend logic."
---

## Relevant Tools

Process model tools cover the full lifecycle:

- **Create a process model** — provide a name, description, parent folder UUID (must be a process model folder), and security group name; optionally include nodes, variables, start form configuration, and type
- **List and get process models** — retrieve process models by UUID or search with optional filtering; scope to an application with `appUuid`
- **Update a process model** — modify name, description, nodes, variables, or connections on an existing process model
- **Delete a process model** — permanently remove a process model by UUID
- **List process model folders** — retrieve top-level process model folders (regular folders cannot contain process models)
- **List application processes** — retrieve runtime process instances for an application, with optional status filtering (RUNNING, COMPLETED, FAILED, CANCELLED)

## Creating Process Models

### Naming Conventions

- **Prefix**: Use the application prefix followed by a space (e.g., `ACME `)
- **Pattern**: `[PREFIX] [Descriptive Action Name]` in Title Case
- **Name should describe the business action**, not the technical implementation
- **Examples**:
  - `ACME Create Case` — process for creating a new case record
  - `ACME Reassign Case` — record action for reassigning a case to a different user
  - `ACME Close Case` — record action for closing a case with resolution notes
  - `ACME Send Notification` — utility process for sending email notifications
  - `ACME Escalate Case` — record action for escalating a case

### Required Parameters

Every process model requires:

- `name` — the process model name following naming conventions
- `description` — what the process does and when it runs
- `parentFolderUuid` — UUID of a process model folder (not a regular folder); discover available folders by listing process model folders
- `securityGroupName` — name of the security group that controls access; typically the application's administrators group

### Minimal Process Model

If you omit the `nodes` parameter, a minimal Start → End flow is created automatically. This is useful when you want to create the process model first and configure nodes later.

### Process Model Type

The `type` parameter defaults to `processModel`. This is the standard type for most workflows.

## Nodes and Flow

### Node Structure

Each node in a process model is defined by:

- `id` — unique integer identifier within the process model (start at 1, increment sequentially)
- `type` — activity class schema ID identifying the node type (e.g., `core.0` for Start Event)
- `coordinates` — `[x, y]` position on the canvas for visual layout
- `connections` — list of target node IDs for outgoing edges (empty list `[]` for End Event)
- `name` — optional display label on the canvas
- `description` — optional description
- `configurations` — optional type-specific configuration (varies by node type)

Load `references/node-types.md` for the complete node type reference with schema IDs, configuration shapes, and examples.

### Flow Design

Nodes connect via the `connections` array. Each entry is the `id` of a target node:

- **Linear flow**: each node connects to exactly one next node
- **Branching (XOR)**: an XOR gateway connects to multiple target nodes; conditions determine which path executes
- **Convergence**: multiple paths can connect to the same target node (merge point)

Every process model must have exactly one Start Event (`core.0`) and at least one End Event (`core.1`). The Start Event is always the entry point; End Events have empty connections (`[]`).

### Canvas Layout

Use coordinates to arrange nodes visually:

- Place nodes left-to-right for the main flow
- Start Event at the left (e.g., `[50, 200]`)
- Space nodes approximately 200px apart horizontally
- For XOR branches, offset paths vertically (e.g., upper path at y=100, lower path at y=300)
- End Events at the right

### Common Flow Patterns

**Simple linear flow** (Start → Task → End):
```
Start (id:1) → Script Task (id:2) → End (id:3)
```

**Form submission flow** (Start → User Input → Write Records → End):
```
Start (id:1) → User Input Task (id:2) → Write Records (id:3) → End (id:4)
```

**Conditional flow** (Start → XOR → Path A / Path B → End):
```
Start (id:1) → XOR (id:2) → Script Task A (id:3) → End (id:5)
                            → Script Task B (id:4) → End (id:5)
```

**Form with cancel** (Start → User Input → XOR → Write / Skip → End):
```
Start (id:1) → User Input (id:2) → XOR (id:3) → Write Records (id:4) → End (id:5)
                                                → End (id:5)  [cancel path]
```

## Node Types

### Tooled Node Types

These node types are fully supported when creating or updating process models:

| Schema ID | Name | Purpose |
|---|---|---|
| `core.0` | Start Event | Entry point — exactly one per process model |
| `core.1` | End Event | Termination point — connections must be `[]` |
| `core.4` | XOR (Exclusive Gateway) | Conditional branching — evaluates conditions to choose one path |
| `internal.16` | Script Task | Executes expressions and assigns output variables |
| `internal.17` | User Input Task | Presents an interface form to a user and collects input |
| `internal3.write_records_to_source_23r3` | Write Records | Writes a record to the data source |
| `internal3.sendemail3` | Send E-Mail | Sends an email to specified recipients |

### Untooled Node Types (Bridge to Manual Capabilities)

These node types can be included in process model definitions but represent capabilities that don't have dedicated MCP tools. The agent can create process models with these nodes, but the underlying AI skills, integrations, or database queries must be configured separately in the Appian environment:

| Schema ID | Name | Capability |
|---|---|---|
| `internal3.rs2_ai_skill_document_classification3` | Classify Documents | AI skill — document classification |
| `internal3.rs2_ai_skill_email_classification2` | Classify Emails | AI skill — email classification |
| `internal3.rs2_ai_skill_document_extraction23r4` | Extract from Document | AI skill — document data extraction |
| `internal.database601` | Query Database | Direct database query execution |
| `internal3.integration` | Call Integration | Invokes an integration / connected system |

When a workflow requires an untooled node type, include it in the process model definition and note to the developer that the underlying capability (AI skill, integration, or database query) needs manual configuration.

## Process Variables

Process variables store data that flows through the process. Each variable needs:

- `name` — camelCase identifier (e.g., `caseRecord`, `isApproved`, `assignedUser`)
- `type` — one of: `TEXT`, `INTEGER`, `NUMBER`, `BOOLEAN`, `DATE`, `DATETIME`, `TIME`, `USER`, `GROUP`, `DOCUMENT`, `FOLDER`
- `isParameter` — (optional) `true` if this variable is an input/output parameter visible to callers
- `isRequired` — (optional) `true` if callers must provide this parameter
- `recordTypeUuid` — (optional) UUID of a record type, when the variable holds record data

### Variable Naming

- Use camelCase: `caseRecord`, `isUpdate`, `cancelFlag`
- Name after what the variable represents: `assignedUser` not `userVar`
- Boolean variables use question-style names: `isApproved`, `isCancelled`, `shouldNotify`

### Common Variable Patterns

**Record create/update process**:
- `record` (type matching the record type, `isParameter: true`, `isRequired: false`, `recordTypeUuid` set) — the record data
- `isUpdate` (BOOLEAN, `isParameter: true`) — controls create vs. update mode
- `cancel` (BOOLEAN) — set by the form's Cancel button

**Notification process**:
- `recipientUser` (USER, `isParameter: true`, `isRequired: true`) — who to notify
- `messageBody` (TEXT, `isParameter: true`) — notification content
- `subject` (TEXT, `isParameter: true`) — email subject line

## Start Forms

### Connecting an Interface as a Start Form

To use an interface as the process model's start form, provide `interfaceInformation` when creating the process model:

```
interfaceInformation: {
  name: "ACME_CaseForm",
  uuid: "<interface-uuid>",
  ruleInputs: [
    { name: "record", typeQName: "<record-type-qname>", value: "pv!record" },
    { name: "isUpdate", typeQName: "{http://www.appian.com/ae/types/2009}Boolean", value: "false" },
    { name: "cancel", typeQName: "{http://www.appian.com/ae/types/2009}Boolean", value: "pv!cancel" }
  ]
}
```

Each `ruleInput` maps an interface input to a process variable or literal value:

- `name` — must match the interface input name exactly
- `typeQName` — the Appian type QName for this input
- `value` — a process variable reference (`pv!variableName`) or a literal value

### Start Form Expression

Alternatively, use `startFormExpr` to specify the start form as a SAIL expression:

```
startFormExpr: "=rule!ACME_CaseForm(record: pv!record, isUpdate: false, cancel: pv!cancel)"
```

This is useful when you need to pass computed values or when the interface call requires expression logic.

## Security

### Security Group

Every process model requires a `securityGroupName` — the name of an Appian group that controls who can view and modify the process model. Typically this is the application's administrators group (e.g., `ACME Administrators`).

### Process Model Folders

Process models must live in process model folders, not regular folders. Use the process model folder listing tool to discover available folders. The `parentFolderUuid` must reference a process model folder.

When creating a new application, the application creation typically generates a default process model folder. Discover it by listing process model folders scoped to the application.

## Updating Process Models

### Read Before Update

Always retrieve the current process model before updating. The update replaces provided fields entirely:

1. Get the process model to see its current nodes, variables, and configuration
2. Merge your changes with the existing state
3. Send the complete updated configuration

### Adding or Modifying Nodes

To add nodes to an existing process model:

1. Get the current process model to see existing nodes
2. Add new nodes with unique IDs (don't reuse existing IDs)
3. Update connections on existing nodes to link to the new nodes
4. Send the complete node list (existing + new)

### Updating Variables

To add or modify variables, retrieve the current variable list, make changes, and send the complete list.

## Common Pitfalls

- **Using a regular folder as parentFolderUuid** — process models require process model folders; list process model folders to find valid parent folders
- **Forgetting securityGroupName** — this is required; use the application's administrators group
- **Duplicate node IDs** — each node must have a unique integer ID within the process model; reusing IDs causes unpredictable behavior
- **End Event with connections** — End Events must have `connections: []`; adding connections to an End Event is invalid
- **Missing Start Event** — every process model needs exactly one Start Event (`core.0`)
- **XOR without conditions** — an XOR gateway needs `configurations.conditions` to evaluate which path to take; without conditions, the gateway has no branching logic
- **Mismatched interface input names in start form** — the `ruleInputs` names in `interfaceInformation` must exactly match the interface's input names
- **Updating nodes without including existing ones** — the update replaces the entire node list; always include all existing nodes plus your changes
- **Not creating the interface before referencing it** — if using a start form, the interface must exist before the process model references it
- **Wrong variable type for record data** — when a variable holds record data, set `recordTypeUuid` to the record type's UUID so Appian knows the data shape
- **Forgetting cancel handling** — forms with a Cancel button need an XOR gateway after the User Input Task to check the cancel variable and skip the write step
- **Using plain-text record type names in expressions** — Script Task `outputVariables`, `expressionBody`, and XOR `condition` expressions submitted through the API must use UUID-qualified record type references wrapped in single quotes: `'recordType!{rtUuid}Name.fields.{fieldUuid}fieldName'`. Plain-text names like `recordType!Board Committee Submission.fields.status` fail at parse time, especially when names contain spaces. Retrieve UUIDs from `createRecordType`/`getRecordType` and `listRecordTypeFields`.

## Expression and SAIL Guidance

Process models use Appian expressions in several places: Script Task expression bodies, XOR gateway conditions, and start form expressions. For expression syntax, operators, type system, and common patterns, activate the expressions knowledge skill. For SAIL syntax used in interfaces connected as start forms or User Input Tasks, activate the SAIL knowledge skill.

## When You Need More

Load `references/node-types.md` when you need the complete node type reference with schema IDs, configuration shapes, and examples for each node type.

For questions about specific Appian process model behavior, advanced workflow patterns, or platform capabilities beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.
