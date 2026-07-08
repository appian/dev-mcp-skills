# Process Models

## Table of Contents

- [Update JSON Schema](#update-json-schema)
- [Naming Conventions](#naming-conventions)
- [Required Configuration](#required-configuration)
- [Process Variables](#process-variables)
- [Start Forms](#start-forms)
- [Nodes and Flow](#nodes-and-flow)
- [Node Configuration Patterns](#node-configuration-patterns)
- [Flow Patterns](#flow-patterns)
- [Complete Example](#complete-example)
- [Security](#security)
- [Updating Process Models](#updating-process-models)
- [Common Pitfalls](#common-pitfalls)

## Update JSON Schema

```json
{
  "description": "Creates a new case record from the submitted form",
  "processVariables": [
    {"name": "record", "type": "<typeReference from record type details>", "isParameter": true, "isRequired": false},
    {"name": "isUpdate", "type": "BOOLEAN", "isParameter": true},
    {"name": "cancel", "type": "BOOLEAN", "isParameter": true}
  ],
  "nodes": [
    {"id": 1, "type": "core.0", "name": "Start", "coordinates": [50, 200], "connections": [2]},
    {"id": 2, "type": "core.4", "name": "Was Cancelled?", "coordinates": [250, 200], "connections": [3, 4]},
    {"id": 3, "type": "internal3.write_records_to_source_23r3", "name": "Write Case", "coordinates": [450, 100], "connections": [4]},
    {"id": 4, "type": "core.1", "name": "End", "coordinates": [650, 200], "connections": []}
  ],
  "startForm": {
    "interfaceUuid": "<interface-uuid>",
    "inputMap": {"record": "record", "isUpdate": "isUpdate", "cancel": "cancel"}
  }
}
```

## Naming Conventions

- **Prefix**: Application prefix + space (e.g., `CM `)
- **Pattern**: `PREFIX ActionName` in Title Case
- **Name describes the business action**
- **Examples**:
  - `CM Create Case`
  - `CM Reassign Case`
  - `CM Close Case`
  - `CM Send Notification`

## Required Configuration

Every process model needs:
- `name` — follows naming conventions
- `description` — what it does and when it runs
- `parentFolderUuid` — a process model folder (not a regular folder). Discover from the application's `defaultObjects.processModelFolderUuid`.
- `errorAlertGroupName` — group controlling error alerts (typically PREFIX Administrators)

## Process Variables

Each variable:
- `name` — camelCase (`caseRecord`, `isApproved`)
- `type` — built-in: Text, Number (Integer), Number (Decimal), Boolean, Date, Time, Date and Time, User, Group, Document, Folder. For record types: use the `typeReference` string from the record type's details
- `isParameter` (optional) — `true` = visible to callers
- `isRequired` (optional) — callers must provide
- `multiple` (optional) — holds a list

### Common Variable Patterns

**Record create/update:**
- `record` (type: typeReference from record type details, isParameter: true)
- `isUpdate` (BOOLEAN, isParameter: true)
- `cancel` (BOOLEAN)

**Notification:**
- `recipientUser` (USER, isParameter: true, isRequired: true)
- `messageBody` (TEXT, isParameter: true)
- `subject` (TEXT, isParameter: true)

## Start Forms

Connect an interface as a start form using `startForm`:

```json
{
  "startForm": {
    "interfaceUuid": "<uuid>",
    "inputMap": {
      "record": "record",
      "isUpdate": "isUpdate",
      "cancel": "cancel"
    }
  }
}
```

`inputMap` keys = process variable names (without `pv!`), values = interface input names (without `ri!`).

## Nodes and Flow

### Node Fields

Every node has:
- `id` — unique integer within the PM (start at 1, increment)
- `type` — schema ID
- `name` — display label
- `coordinates` — `[x, y]` canvas position
- `connections` — list of target node IDs (empty for End Event)

Activity nodes also require:
- `assignment` — `{"attended": true}` or `{"attended": false}`. Required on all activity nodes.
- `data` — inputs and outputs (schema-defined and custom)
- `forms` — interface config (for attended nodes only)

Gateway nodes support:
- `decision` — conditions and default path

### Common Schema IDs

| Schema ID | Name | Purpose |
|---|---|---|
| `core.0` | Start Event | Entry point (exactly one) |
| `core.1` | End Event | Termination (connections must be `[]`) |
| `core.4` | XOR Gateway | Conditional branching |
| `internal.16` | Script Task | Execute expressions, assign variables |
| `internal.17` | User Input Task | Present form, collect input |
| `internal3.write_records_to_source_23r3` | Write Records | Write record to data source |
| `internal3.sendemail3` | Send E-Mail | Send email |
| `internal3.integration` | Call Integration | Invoke an integration |

### Discovering Node Type Schemas

Use the node type discovery tools to list all available node types and get the input/output schema for a specific type. When working with attended nodes, provide the form interface UUID to discover form inputs. When working with integrations or subprocesses, provide the reference UUID for schema enrichment.

Discover exact input names and required fields before configuring unfamiliar node types. The schema ID table above covers common types — use the node type listing tool for the full catalog.

## Node Configuration Patterns

Four distinct patterns for how nodes are configured:

### Gateway pattern: `decision`

XOR (`core.4`) uses the `decision` field for conditional branching:

```json
{
  "id": 3, "type": "core.4", "name": "Was Cancelled?",
  "coordinates": [450, 200], "connections": [4, 5],
  "decision": {
    "conditions": [
      {"expression": "=pv!cancel", "targetNodeId": 5}
    ],
    "defaultPath": 4
  }
}
```

### Unattended activity pattern: `assignment` + `data.outputs`

Script Task (`internal.16`) — schema returns empty inputs/outputs, use `data.outputs` with `expression`/`saveInto`:

```json
{
  "id": 2, "type": "internal.16", "name": "Set Defaults",
  "coordinates": [250, 200], "connections": [3],
  "assignment": {"attended": false},
  "data": {
    "outputs": [
      {"expression": "=\"Open\"", "saveInto": "pv!status"},
      {"expression": "=today()", "saveInto": "pv!createdDate"}
    ]
  }
}
```

### Attended activity pattern: `assignment` + `forms`

User Input Task (`internal.17`) — uses `forms` with interface reference:

```json
{
  "id": 2, "type": "internal.17", "name": "Employee Form",
  "coordinates": [250, 200], "connections": [3],
  "assignment": {"attended": true},
  "forms": {
    "interfaceUuid": "<interface-uuid>",
    "inputMap": {"employee": "employee", "cancel": "cancel"}
  }
}
```

`inputMap` keys = interface input names (without `ri!`), values = process variable names (without `pv!`).

### Schema-defined inputs pattern: `assignment` + `data.inputs`

Write Records (`internal3.write_records_to_source_23r3`) — inputs discovered via the node type schema:

```json
{
  "id": 4, "type": "internal3.write_records_to_source_23r3", "name": "Write Employee",
  "coordinates": [650, 200], "connections": [5],
  "assignment": {"attended": false},
  "data": {
    "inputs": [
      {"name": "Records", "expression": "={pv!employee}"},
      {"name": "Version", "value": 6},
      {"name": "CaptureEvents", "value": false}
    ]
  }
}
```

Note: `Records` wraps in curly braces to create a list. Discover input names with the node type schema tool.

## Flow Patterns

**Linear** (Start → Task → End):
```
Start(1) → Script(2) → End(3)
```

**Start form with write** (Start → Write → End):
When `startForm` is configured, the form is presented at process initiation and populates process variables directly. The process is unattended after that — no User Input Task needed.
```
Start(1) → WriteRecords(2) → End(3)
```

**Start form with cancel** (Start → XOR → Write/End):
```
Start(1) → XOR(2) → WriteRecords(3) → End(4)
                   → End(4)  [cancel path]
```

**Mid-process form** (attended task within a running process):
Use User Input Task when collecting input mid-process from an assigned user.
```
Start(1) → Script(2) → UserInput(3) → WriteRecords(4) → End(5)
```

### Canvas Layout

- Left to right for main flow
- Start at `[50, 200]`
- ~200px horizontal spacing
- XOR branches: offset vertically (upper y=100, lower y=300)

## Complete Example: Form → Cancel Check → Write

```json
{
  "processVariables": [
    {"name": "employee", "type": "<typeReference from record type details>", "isParameter": true},
    {"name": "cancel", "type": "Boolean", "isParameter": true}
  ],
  "nodes": [
    {
      "id": 1, "type": "core.0", "name": "Start",
      "coordinates": [50, 200], "connections": [2]
    },
    {
      "id": 2, "type": "internal.17", "name": "Employee Form",
      "coordinates": [250, 200], "connections": [3],
      "assignment": {"attended": true},
      "forms": {
        "interfaceUuid": "<form-interface-uuid>",
        "inputMap": {"employee": "employee", "cancel": "cancel"}
      }
    },
    {
      "id": 3, "type": "core.4", "name": "Was Cancelled?",
      "coordinates": [450, 200], "connections": [4, 5],
      "decision": {
        "conditions": [{"expression": "=pv!cancel", "targetNodeId": 5}],
        "defaultPath": 4
      }
    },
    {
      "id": 4, "type": "internal3.write_records_to_source_23r3", "name": "Write Employee",
      "coordinates": [650, 200], "connections": [5],
      "assignment": {"attended": false},
      "data": {
        "inputs": [
          {"name": "Records", "expression": "={pv!employee}"},
          {"name": "Version", "value": 6},
          {"name": "CaptureEvents", "value": false}
        ]
      }
    },
    {
      "id": 5, "type": "core.1", "name": "End",
      "coordinates": [850, 200], "connections": []
    }
  ]
}
```

## Security

- `securityGroupName` required — use the application's administrators group
- Process models live in PM folders only (not regular folders)
- Discover PM folders: look at app structure after creation

## Updating Process Models

**Always get before update** — node/variable lists replace entirely. Get the current state, modify what you need, then update with the full payload.

## Common Pitfalls

- **PM in regular folder** — must be in a process model folder
- **Duplicate node IDs** — each must be unique within the PM
- **End Event with connections** — must have `connections: []`
- **Missing Start Event** — every PM needs exactly one `core.0`
- **XOR without conditions** — needs `decision.conditions` for branching logic
- **Mismatched interface input names in start form** — `inputMap` values must match interface inputs exactly
- **Updating without existing nodes** — replaces entire node list; include all existing nodes
- **Interface must exist before referencing** — create interface before PM references it
- **Forgetting cancel handling** — forms with Cancel need XOR after User Input to check cancel variable
- **Plain-text record type names in expressions** — use UUID-qualified format
