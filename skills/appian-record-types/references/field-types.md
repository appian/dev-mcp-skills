# Record Type Field Types

Twelve field types are available for record type source configuration fields. Use the exact `fieldType` string value — these are case-sensitive.

## Field Type Reference

### TEXT

String value for names, descriptions, codes, and free-text content.

- **Constraints**: `length` (integer, default 0 = unlimited). Set an appropriate max length for database column sizing — unlimited length uses a LOB column type which has query limitations.
- **When to use**: Names, titles, descriptions, codes, notes, any textual content. Use for short codes on reference tables (max 20 chars) and display names (max 100 chars).
- **Naming**: No special prefix. Use `_code` suffix for short lookup codes, `_name` suffix for display names.
- **Example**: `fieldName: "case_title", fieldType: "TEXT", length: 200`

### NUMBER

Numeric value that can hold both integers and decimals. A general-purpose numeric type.

- **Constraints**: None specific. Stores both whole numbers and decimals.
- **When to use**: When you need a numeric field that may hold either integers or decimals, or when the precision requirements are not strict. Prefer INTEGER or DECIMAL when the data type is known.
- **Example**: `fieldName: "score", fieldType: "NUMBER"`

### INTEGER

Whole number value without decimals.

- **Constraints**: None specific. No decimal component.
- **When to use**: Primary keys (with `isPrimaryKey: true`), foreign keys, counts, quantities, and any whole-number value. Every record type must have exactly one INTEGER primary key field.
- **Naming**: Primary keys use `[table_name_lowercase]_id`. Foreign keys use `[referenced_table_lowercase]_id`.
- **Example**: `fieldName: "case_id", fieldType: "INTEGER", isPrimaryKey: true`

### DECIMAL

Numeric value with decimal precision.

- **Constraints**: None specific. Stores fractional values.
- **When to use**: Currency amounts, percentages, measurements, ratings, and any value requiring decimal precision.
- **Example**: `fieldName: "total_amount", fieldType: "DECIMAL"`

### DATE

Date value without a time component (year-month-day).

- **Constraints**: None specific. Stores date only.
- **When to use**: Birthdays, due dates, effective dates, and any date where time of day is not relevant.
- **Naming**: Use `_on` suffix for date fields (e.g., `due_on`, `effective_on`). Use `_date` suffix when the field name already implies a date context (e.g., `birth_date`).
- **Example**: `fieldName: "due_on", fieldType: "DATE"`

### DATETIME

Date and time value with timestamp precision.

- **Constraints**: None specific. Stores both date and time.
- **When to use**: Audit timestamps (created_at, updated_at), event timestamps, and any value where both date and time matter.
- **Naming**: Use `_at` suffix for timestamp fields (e.g., `created_at`, `closed_at`).
- **Example**: `fieldName: "created_at", fieldType: "DATETIME"`

### TIME

Time value without a date component.

- **Constraints**: None specific. Stores time of day only.
- **When to use**: Schedules, operating hours, recurring time-based events where the date is not relevant.
- **Example**: `fieldName: "start_time", fieldType: "TIME"`

### BOOLEAN

True/false value.

- **Constraints**: None specific. Stores true or false.
- **When to use**: Flags, toggles, binary states. Use for universal binary concepts (active/inactive, enabled/disabled). For domain-specific binary pairs (approved/rejected), consider a reference table instead.
- **Naming**: Use `is_` prefix (e.g., `is_active`, `is_archived`, `is_primary`).
- **Example**: `fieldName: "is_active", fieldType: "BOOLEAN"`

### USER

Reference to an Appian user.

- **Constraints**: None specific. Stores an Appian username.
- **When to use**: Assigned users, created-by/modified-by audit fields, and any field that references an Appian platform user. Never create a separate record type to represent users — use USER fields that reference Appian's built-in user system.
- **Naming**: Use `_username` suffix for user reference fields (e.g., `assigned_username`, `created_by_username`).
- **Example**: `fieldName: "assigned_username", fieldType: "USER"`

### GROUP

Reference to an Appian group.

- **Constraints**: None specific. Stores an Appian group identifier.
- **When to use**: When a record needs to reference an Appian security group — for example, assigning ownership to a team or department group.
- **Example**: `fieldName: "owning_group", fieldType: "GROUP"`

### DOCUMENT

Reference to an Appian document.

- **Constraints**: None specific. Stores a reference to an uploaded document.
- **When to use**: File attachments, uploaded documents. For a single document per record, add a DOCUMENT field directly. For multiple documents per record, create a separate child record type with a DOCUMENT field and a one-to-many relationship from the parent.
- **Example**: `fieldName: "attachment", fieldType: "DOCUMENT"`

### FOLDER

Reference to an Appian folder.

- **Constraints**: None specific. Stores a reference to an Appian folder.
- **When to use**: When a record needs to reference a specific folder — for example, a project record that has an associated document folder.
- **Example**: `fieldName: "document_folder", fieldType: "FOLDER"`

## Field Configuration Properties

Each field in `sourceAndCustomFields` supports these properties:

| Property | Type | Required | Default | Description |
|---|---|---|---|---|
| `fieldName` | string | yes | — | Column name in snake_case |
| `fieldType` | string | yes | — | One of the 12 types above |
| `isPrimaryKey` | boolean | no | false | Exactly one field per record type must be true |
| `isUnique` | boolean | no | false | Enforces uniqueness constraint on the column |
| `length` | integer | no | 0 | Column length for TEXT fields (0 = unlimited) |
| `displayName` | string | no | — | Human-readable label for the field |
| `description` | string | no | — | Description of the field's purpose |

## Quick Reference Table

| fieldType | Description | Key Constraint | Naming Convention |
|---|---|---|---|
| TEXT | String value | `length` for sizing | `_code`, `_name` suffixes |
| NUMBER | General numeric | — | — |
| INTEGER | Whole number | Use for PK/FK | `_id` suffix for keys |
| DECIMAL | Decimal number | — | — |
| DATE | Date only | — | `_on` or `_date` suffix |
| DATETIME | Date and time | — | `_at` suffix |
| TIME | Time only | — | `_time` suffix |
| BOOLEAN | True/false | — | `is_` prefix |
| USER | Appian user ref | — | `_username` suffix |
| GROUP | Appian group ref | — | — |
| DOCUMENT | Document ref | — | — |
| FOLDER | Folder ref | — | — |
