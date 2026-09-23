# Record Events

Record events track business-level changes on your record types, enabling activity logs, auditing, collaboration, and process mining in Process HQ.

## When to Use Record Events

Configure record events on record types that represent **major business entities or processes** — the things your app manages end-to-end. Good candidates: Cases, Orders, Applications, Accounts, Employees.

Do **not** configure record events on lookup/reference record types (Regions, Statuses, Categories) — they store static data that rarely changes and does not represent a process.

### Use Cases

- **Activity log / audit trail** — show who did what and when on a record
- **Collaboration** — let users comment and reply directly on record events via the Event History List component
- **Process mining** — feed event data into Process HQ for process insights, conformance analysis, and duration calculations
- **Automation measurement** — track which events were completed by humans vs. automation (RPA, AI, robotic tasks)

---

## Configure JSON Schema

```json
{
  "eventTypes": ["Created Order", "Reviewed Order", "Approved Order", "Shipped Order"]
}
```

The `eventTypes` array is optional. If omitted, Appian generates default common event types (Created, Updated, Commented On). Provide event types when you want to track specific business lifecycle events.

### Configure Response

```json
{
  "eventHistoryRecordTypeUuid": "uuid-1",
  "eventTypeLookupRecordTypeUuid": "uuid-2",
  "replyThreadRecordTypeUuid": "uuid-3",
  "subscriberRecordTypeUuid": "uuid-4",
  "eventRelationshipUuid": "uuid-rel-1",
  "eventTypeRelationshipUuid": "uuid-rel-2",
  "eventReplyRelationshipUuid": "uuid-rel-3",
  "subscriberRelationshipUuid": "uuid-rel-4",
  "eventTypeValueFieldUuid": "uuid-field-1",
  "eventUserFieldUuid": "uuid-field-2",
  "eventTimestampFieldUuid": "uuid-field-3",
  "eventAutomationIdentifierFieldUuid": "uuid-field-4",
  "eventCommentFieldUuid": "uuid-field-5",
  "eventReplyUserFieldUuid": "uuid-field-6",
  "eventReplyCommentFieldUuid": "uuid-field-7",
  "eventReplyTimestampFieldUuid": "uuid-field-8",
  "subscriberUserFieldUuid": "uuid-field-9",
  "commentEventTypeId": 3,
  "generateCommonEvents": true,
  "eventTypes": [
    { "id": 1, "value": "Created Order" },
    { "id": 2, "value": "Reviewed Order" },
    { "id": 3, "value": "Commented On Order" },
    { "id": 4, "value": "Approved Order" },
    { "id": 5, "value": "Shipped Order" }
  ]
}
```

---

## Generated Record Types

Configuring record events generates four supporting record types:

| Record Type | Purpose |
|---|---|
| **Event History** | Stores who completed an event and when. Core fields: event type, user, timestamp, automation identifier, comment. |
| **Event Type Lookup** | Static lookup of possible event types (e.g., "Created Order", "Shipped Order"). |
| **Reply Thread** | Stores threaded conversation replies on events (for collaboration). |
| **Subscriber** | Stores users subscribed to event notifications. |

These record types are automatically:
- Created with appropriate fields and relationships
- Wired to the base record type via relationships
- Given the same security as the base record type

---

## Event Type Design Guidelines

### Choose events that represent process steps

Track events that represent the main steps in your business process. Each event should ideally occur **once per record** and events should occur in a **specific order**. This provides optimal data for Process HQ analysis.

**Good event types** (Financial Onboarding):
- Received Application
- Completed Pre-Onboarding Compliance Check
- Completed Know Your Customer (KYC) Review
- Completed Financial Document Verification
- Completed Financial Review
- Approved Application
- Completed User Account Provisioning
- Funded Account

### Use past-tense verb phrases

Since each event represents a completed activity, name event types in past tense.

✅ `Created Order`, `Approved Application`, `Shipped Package`
❌ `Create Order`, `Approval`, `Shipping`

### Avoid status names

Do not use statuses like "Pending", "In Progress", or "In Review" as event types. These reflect state, not the action that changed the state.

✅ `Verified Documents`, `Assigned Onboarding Manager`, `Completed Review`
❌ `Pending`, `In Progress`, `In Review`

### Avoid dynamic details in event type names

Capture dynamic details (who, how much, which version) in record fields, not in the event type name.

✅ Event type: `Completed Financial Review` + user field stores "Angela Lewis"
❌ Event type: `Financial Review Completed by Angela Lewis`

### Distinguish ad-hoc events from process events

- **Process events** — specific, ordered lifecycle steps (best for Process HQ)
- **Ad-hoc events** — comments, updates that can happen at any time and any number of times

If you want both activity logs and process mining, track both but configure Process HQ to focus only on process events.

---

## Writing Events in Process Models

Events are written via the **Write Records** smart service node (`internal3.write_records_to_source_23r3`) in process models. When a record type has record events configured, the Write Records node gains event-writing capabilities.

### Key points

- Events can only be written via the Write Records node in a process model — `a!writeRecords()` function does not support event writing
- You can only write events for **one record type per Write Records node**
- If you generate Create or Update record actions, Appian auto-configures Write Records to write events
- For custom events (e.g., "Reviewed Order", "Shipped Order"), manually configure the Write Records node

### Write Records node inputs for events

| Input | Type | Required | Description |
|---|---|---|---|
| `Records` | Any Type (list) | Yes | The records to write. Wrap in braces: `={pv!record}` |
| `Version` | Number (Integer) | Yes | Always set `value=6` |
| `RecordType` | RecordType | Yes (for events) | The record type to capture events for. Format: `={'recordType!{UUID}Name'}` |
| `CaptureEvents` | Boolean | Yes | `true` = always capture, `false` = never. Use expression for conditional (e.g., `pv!shouldCapture`) |
| `EventType` | Number (Integer) | Yes (when capturing) | Event type ID from the Event Type Lookup table |
| `User` | User | No | User to associate with the event. Default: `pp!initiator` |
| `Timestamp` | Date and Time | No | When the event occurred. Default: `now()` |
| `AutomationType` | Number (Integer) | No | Automation type. Use `a!automationId()` with: NONE, RPA, AI, INTEGRATION, OTHER |
| `comment` | Text | No | Optional comment for the event |

### Example: Write Records node with events in a process model

```json
{
  "nodes": [
    {"id": 1, "type": "core.0", "name": "Start", "coordinates": [50, 200], "connections": [2]},
    {
      "id": 2,
      "type": "internal3.write_records_to_source_23r3",
      "name": "Write and Capture Event",
      "coordinates": [250, 200],
      "connections": [3],
      "assignment": {"attended": false},
      "data": {
        "inputs": [
          {"name": "Records", "expression": "{pv!record}"},
          {"name": "Version", "value": 6},
          {"name": "RecordType", "expression": "'recordType!{rt-uuid}Order'"},
          {"name": "CaptureEvents", "value": true},
          {"name": "EventType", "value": 4},
          {"name": "AutomationType", "expression": "a!automationId(\"NONE\")"}
        ],
        "outputs": [
          {"name": "RecordsUpdated", "saveInto": "pv!updatedRecord"}
        ]
      }
    },
    {"id": 3, "type": "core.1", "name": "End", "coordinates": [450, 200], "connections": []}
  ]
}
```

### Finding event type IDs

Event type IDs are stored in the Event Type Lookup record type. To find them:
1. Get the record events configuration for your record type
2. Extract the `eventTypeLookupRecordTypeUuid` from the response
3. List the record data on that lookup record type to see event type IDs and values

### Conditional event capture

Use an expression for `CaptureEvents` to conditionally write events:

```json
{"name": "CaptureEvents", "expression": "pv!isNewRecord"}
```

---

## Tracking Automation

Record events can track whether an event was completed by a human or by automation (RPA, AI Skills, robotic tasks). This enables automation measurement in Process HQ.

### Setup

The Event History record type includes an `automationTypeId` field (Number Integer) when generated. This field stores a standard automation type identifier.

### Standard automation type IDs

| ID | Automation Type |
|---|---|
| 1 | Robotic Task |
| 2 | Appian RPA |
| 3 | AI Skill |

When configuring the Write Records node for automated steps, set the **Automation Type** to the appropriate value. Leave it empty for human-completed events.

### Process HQ integration

Once automation types are captured:
- Map the automation identifier field as the **Automation** attribute when adding a process in Process HQ
- Automation statistics appear in the Conformance and Costs tab
- Use the Automation attribute for custom KPIs

---

## Displaying Events in Interfaces

Use `a!eventHistoryListField()` to display event history in record views, dashboards, and site pages.

### Event styles

| Style | Value | Best for |
|---|---|---|
| Preview List | `"PREVIEW_LIST"` | Record views alongside other content. Shows short list with "view all" dialog. |
| Full List | `"FULL_LIST"` | Dedicated event page. Searchable, filterable. Supports comments and threads. |
| Comment List | `"LIST_WITH_COMMENTS"` | Collaboration-heavy views. Unified activity stream with comments and replies. |
| Timeline | `"TIMELINE"` | Time-based analysis. Events organized chronologically. |

### a!eventData() parameters

| Parameter | Description |
|---|---|
| `recordType` | The Event History record type |
| `filters` | Optional filter expression (or null for all events on the record) |
| `timestamp` | Field reference to the timestamp field |
| `user` | Field reference to the user field |
| `eventTypeName` | Expression for the event type label — typically traverse the relationship: `fv!data[recordType!EventHistory.relationships.eventType.fields.eventName]` |
| `details` | Optional expression for additional event details |
| `recordTypeForTag` | The base record type (for record link tags) |
| `recordIdentifier` | Field reference to the FK linking events to the base record |
| `allowUsersToComment` | `true` to enable top-level comments |
| `allowUsersToReply` | `true` to enable threaded replies |
| `baseRecordIdForComment` | Expression for the base record ID (required when comments enabled): `ri!record[recordType!Order.fields.id]` |
| `comment` | Field reference to the comment field on Event History |

### Example: Preview list (record view)

```sail
a!eventHistoryListField(
  label: "Event History",
  labelPosition: "COLLAPSED",
  eventData: {
    a!eventData(
      recordType: recordType!{ehUuid}Order Event History,
      filters: a!queryFilter(
        field: recordType!{ehUuid}Order Event History.fields.{fkFieldUuid}orderId,
        operator: "=",
        value: ri!record[recordType!{rtUuid}Order.fields.{pkUuid}id]
      ),
      timestamp: recordType!{ehUuid}Order Event History.fields.{tsUuid}timestamp,
      user: recordType!{ehUuid}Order Event History.fields.{userUuid}user,
      eventTypeName: fv!data[recordType!{ehUuid}Order Event History.relationships.{relUuid}eventType.fields.{nameUuid}value],
      recordTypeForTag: recordType!{rtUuid}Order,
      recordIdentifier: recordType!{ehUuid}Order Event History.fields.{fkFieldUuid}orderId
    )
  },
  refreshAfter: "RECORD_ACTION",
  eventStyle: "PREVIEW_LIST",
  previewListPageSize: 5,
  pageSize: 10,
  formatTimestamp: "DATE_TIME",
  displayUser: "IMAGE",
  showSearchBox: true,
  eventFilters: {"EVENT_TYPE", "DATE_RANGE"},
  showReverseSortButton: true
)
```

### Example: Full list with comments and collaboration

```sail
a!eventHistoryListField(
  label: "Activity",
  labelPosition: "COLLAPSED",
  eventData: {
    a!eventData(
      recordType: recordType!{ehUuid}Order Event History,
      filters: a!queryFilter(
        field: recordType!{ehUuid}Order Event History.fields.{fkFieldUuid}orderId,
        operator: "=",
        value: ri!record[recordType!{rtUuid}Order.fields.{pkUuid}id]
      ),
      timestamp: recordType!{ehUuid}Order Event History.fields.{tsUuid}timestamp,
      user: recordType!{ehUuid}Order Event History.fields.{userUuid}user,
      comment: recordType!{ehUuid}Order Event History.fields.{commentUuid}comment,
      eventTypeName: fv!data[recordType!{ehUuid}Order Event History.relationships.{relUuid}eventType.fields.{nameUuid}value],
      details: if(
        a!isNullOrEmpty(fv!data[recordType!{ehUuid}Order Event History.fields.{userUuid}user]),
        a!automationType(fv!data[recordType!{ehUuid}Order Event History.fields.{autoUuid}automationTypeId]) & " completed this step.",
        {}
      ),
      recordTypeForTag: recordType!{rtUuid}Order,
      recordIdentifier: recordType!{ehUuid}Order Event History.fields.{fkFieldUuid}orderId,
      allowUsersToComment: true,
      allowUsersToReply: true,
      baseRecordIdForComment: ri!record[recordType!{rtUuid}Order.fields.{pkUuid}id]
    )
  },
  refreshAfter: "RECORD_ACTION",
  eventStyle: "FULL_LIST",
  pageSize: 10,
  formatTimestamp: "DATE_TIME",
  collapseDetailsByDefault: true,
  displayUser: "IMAGE",
  showSearchBox: true,
  eventFilters: {"EVENT_TYPE", "DATE_RANGE"},
  showReverseSortButton: true
)
```

### Displaying automation types

Use `a!automationType()` in the `details` parameter to show what automation completed an event:

```sail
details: if(
  a!isNullOrEmpty(fv!data[recordType!{ehUuid}EventHistory.fields.{userUuid}user]),
  a!automationType(fv!data[recordType!{ehUuid}EventHistory.fields.{autoUuid}automationTypeId]) & " completed this step.",
  {}
)
```

---

## Collaboration Features

Record events support collaboration via the Event History List component.

### Top-level comments

- Stored as ad-hoc events in the Event History record type
- Appear inline with other events
- Require the comment field to be mapped in record events configuration
- Enable with `allowUsersToComment: true` in `a!eventData()`

### Threaded conversations

- Stored in the Reply Thread record type (separate from events)
- Appear nested under events and comments
- Require Reply Thread record type to be configured
- Enable with `allowUsersToReply: true` in `a!eventData()`

### Subscribers

- Stored in the Subscriber record type
- Users can subscribe to receive email notifications on new activity
- Mentions (`@user`) trigger email notifications

---

## Process HQ Integration

Record events provide the foundation for process mining in Process HQ. The generated Event History record type already meets the data model requirements:

### Required fields for Process HQ

| Field | Purpose |
|---|---|
| Case ID (FK to base record) | Links events to cases |
| Event Type | The activity performed |
| Timestamp | When the event occurred |
| User | Who performed it |
| Automation ID | Whether it was automated (optional but recommended) |

### Steps to enable process mining

1. Configure record events on your base record type
2. Define event types that represent ordered process steps
3. Write events in your process models via Write Records nodes
4. In Process HQ, add a process and map attributes:
   - **Case ID** → the FK field linking Event History to the base record
   - **Activity** → the event type field
   - **Timestamp** → the timestamp field
   - **User** → the user field (optional)
   - **Automation** → the automation identifier field (optional)

---

## Common Patterns

### Pattern: Configure events on a new record type

1. Create the base record type with fields
2. Configure record events with business lifecycle event types:
```json
{
  "eventTypes": ["Created Case", "Assigned Case", "Reviewed Case", "Escalated Case", "Resolved Case", "Closed Case"]
}
```

### Pattern: Check existing events configuration

Get the record events configuration for the record type. The response includes all field UUIDs, relationship UUIDs, and event type definitions you need for subsequent operations.

### Pattern: Add event types after initial configuration

Event types are stored in the Event Type Lookup record type. To add new event types, insert records into that table:
1. Get the events configuration to find `eventTypeLookupRecordTypeUuid`
2. Insert new rows into the lookup record type using CSV format (column: `value`)

---

## Pitfalls

- **409 Conflict** — `configureRecordEvents` returns 409 if events are already configured. Check the existing configuration first.
- **Event types should be lifecycle-oriented** — avoid generic "Updated" events; instead track specific business actions
- **One record type per Write Records node** — you cannot write events for multiple record types in a single Write Records node
- **a!writeRecords() does not write events** — only the Write Records node in process models supports event writing
- **Don't configure on lookup tables** — record events are for business entities, not static reference data
- **Process HQ requires synced record types** — generated event record types are synced by default, but if you later change the source configuration, process insights will break
