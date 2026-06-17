# Entity Patterns Reference

## Standard Entity Template

Every entity follows this attribute structure. Fields are ordered: primary key → business fields → foreign keys → audit fields.

```yaml
entities:
  - name: "ENTITY_NAME"
    business_purpose: "What this entity tracks and why"
    attributes:
      # 1. Surrogate primary key
      - name: "entity_name_id"
        type: "INTEGER"
        required: true
        primary_key: true
        auto_increment: true
        description: "Unique identifier"

      # 2. Business fields (from requirements)
      - name: "title"
        type: "TEXT"
        required: true
        max_length: 200
        description: "Brief description"

      # 3. Foreign keys to related entities
      - name: "related_entity_id"
        type: "INTEGER"
        required: true
        foreign_key: "RELATED_ENTITY.related_entity_id"
        description: "Reference to related entity"

      # 4. Foreign keys to reference tables
      - name: "case_status_id"
        type: "INTEGER"
        required: true
        foreign_key: "CASE_STATUS.case_status_id"
        description: "Current status"

      # 5. Audit fields (always last)
      - name: "created_by_username"
        type: "USERNAME"
        required: false
        foreign_key: "SYSTEM_USERS.username"
        description: "User who created this record"
      - name: "modified_by_username"
        type: "USERNAME"
        required: false
        foreign_key: "SYSTEM_USERS.username"
        description: "User who last modified this record"
      - name: "created_at"
        type: "DATETIME"
        required: false
        description: "Timestamp when record was created"
      - name: "updated_at"
        type: "DATETIME"
        required: false
        description: "Timestamp when record was last modified"

    relationships:
      - name: "relatedEntity"
        type: "many-to-one"
        entity: "RELATED_ENTITY"
        foreign_key: "related_entity_id"
        references: "related_entity_id"
        description: "Each record belongs to one related entity"
      - name: "caseStatus"
        type: "many-to-one"
        entity: "CASE_STATUS"
        foreign_key: "case_status_id"
        references: "case_status_id"
        description: "Each record has one status"
      - name: "createdBy"
        type: "many-to-one"
        entity: "SYSTEM_USERS"
        foreign_key: "created_by_username"
        references: "username"
        description: "User who created this record"
      - name: "modifiedBy"
        type: "many-to-one"
        entity: "SYSTEM_USERS"
        foreign_key: "modified_by_username"
        references: "username"
        description: "User who last modified this record"
```

## Field Type Reference

| Type | Description | Constraints |
|---|---|---|
| TEXT | String value | `max_length` (default 0 = unlimited). Keep under 30 chars for column name. |
| INTEGER | Whole number | Used for PKs, FKs, counts |
| DECIMAL | Numeric with decimals | Used for currency, percentages, measurements |
| DATE | Date only (year-month-day) | Use `_on` suffix in field name |
| DATETIME | Date and time | Use `_at` suffix in field name |
| BOOLEAN | True/false | Use `is_` prefix in field name |
| USERNAME | Appian user reference | Always set `foreign_key: "SYSTEM_USERS.username"` |
| GROUP | Appian group reference | References an Appian group |
| DOCUMENT | File/document reference | For uploaded files |

## Field Naming Conventions

### Temporal Fields

| Requirements Phrase | Field Name | Type | Rule |
|---|---|---|---|
| "track when created" (no specific name) | `created_at` | DATETIME | `_at` for timestamp |
| "creation date" (no time component) | `created_on` | DATE | `_on` for day-level |
| "track when modified" | `modified_at` | DATETIME | `_at` for timestamp |
| "modification date" | `modified_on` | DATE | `_on` for day-level |
| "track when closed" | `closed_at` | DATETIME | `_at` for timestamp |
| "closure date" | `closed_on` | DATE | `_on` for day-level |
| "capture opened timestamp" | `opened_timestamp` | DATETIME | Exact extraction from source |

Rule: If source says "timestamp" or "time" → DATETIME with `_at`. If source says "date" without "time" → DATE with `_on`. If source uses a specific field name, extract it exactly.

### Deterministic Naming

When extracting field names from requirements, use the exact noun phrase — do not paraphrase, shorten, or expand.

| Source Quote | Wrong | Correct |
|---|---|---|
| "case priority level is High/Medium/Low" | `priority` | `priority_level` |
| "track the case opened date" | `open_date` | `opened_date` |
| "case status code" | `status` | `status_code` |
| "expected closure date" | `closure_date` | `expected_closure_date` |

Preserve qualifiers that provide meaning (code, level, date, score). For similar fields, keep distinctions (e.g., `initial_date` vs. `final_date`).

### Standard Implied Patterns

For audit/tracking fields implied by generic phrases (not explicitly named):

| Requirements Phrase | Standard Field Name | Type |
|---|---|---|
| "track who created", "created by" | `created_by` | USERNAME |
| "track who modified", "modified by" | `modified_by` | USERNAME |
| "track when created" | `created_at` | DATETIME |
| "track when modified" | `modified_at` | DATETIME |

Use implied patterns only when the source quote is generic. If the source provides a specific field name, use exact extraction instead.

## Common Entity Patterns

### Main Transactional Entity

The core business object that users create, update, and track through a lifecycle.

```yaml
- name: "CASE"
  business_purpose: "Tracks customer service cases from creation to resolution"
  attributes:
    - name: "case_id"
      type: "INTEGER"
      primary_key: true
      auto_increment: true
    - name: "title"
      type: "TEXT"
      required: true
      max_length: 200
    - name: "description"
      type: "TEXT"
      required: false
      max_length: 4000
    - name: "case_status_id"
      type: "INTEGER"
      required: true
      foreign_key: "CASE_STATUS.case_status_id"
    - name: "case_priority_id"
      type: "INTEGER"
      required: true
      foreign_key: "CASE_PRIORITY.case_priority_id"
    - name: "assigned_username"
      type: "USERNAME"
      required: false
      foreign_key: "SYSTEM_USERS.username"
    - name: "created_by_username"
      type: "USERNAME"
      required: false
      foreign_key: "SYSTEM_USERS.username"
    - name: "modified_by_username"
      type: "USERNAME"
      required: false
      foreign_key: "SYSTEM_USERS.username"
    - name: "created_at"
      type: "DATETIME"
      required: false
    - name: "updated_at"
      type: "DATETIME"
      required: false
```

### Child Entity (One-to-Many)

An entity that belongs to a parent and grows with user actions (notes, attachments, history entries).

```yaml
- name: "CASE_NOTE"
  business_purpose: "Stores notes added to cases by users"
  attributes:
    - name: "case_note_id"
      type: "INTEGER"
      primary_key: true
      auto_increment: true
    - name: "case_id"
      type: "INTEGER"
      required: true
      foreign_key: "CASE.case_id"
    - name: "note_text"
      type: "TEXT"
      required: true
      max_length: 4000
    - name: "created_by_username"
      type: "USERNAME"
      required: false
      foreign_key: "SYSTEM_USERS.username"
    - name: "modified_by_username"
      type: "USERNAME"
      required: false
      foreign_key: "SYSTEM_USERS.username"
    - name: "created_at"
      type: "DATETIME"
      required: false
    - name: "updated_at"
      type: "DATETIME"
      required: false
```

### Document Entity (Multiple Documents per Record)

When a parent entity needs multiple file uploads, create a separate document table.

```yaml
- name: "CASE_DOCUMENT"
  business_purpose: "Stores documents attached to cases"
  attributes:
    - name: "case_document_id"
      type: "INTEGER"
      primary_key: true
      auto_increment: true
    - name: "case_id"
      type: "INTEGER"
      required: true
      foreign_key: "CASE.case_id"
    - name: "document"
      type: "DOCUMENT"
      required: true
    - name: "document_name"
      type: "TEXT"
      required: true
      max_length: 255
    - name: "created_by_username"
      type: "USERNAME"
      required: false
      foreign_key: "SYSTEM_USERS.username"
    - name: "modified_by_username"
      type: "USERNAME"
      required: false
      foreign_key: "SYSTEM_USERS.username"
    - name: "created_at"
      type: "DATETIME"
      required: false
    - name: "updated_at"
      type: "DATETIME"
      required: false
```

### Instance Entity (No _INSTANCE Suffix)

For workflow or process instances, use the base noun without suffixes.

```yaml
# Correct: WORKFLOW (not WORKFLOW_INSTANCE)
- name: "WORKFLOW"
  business_purpose: "Tracks workflow instances for case processing"

# Correct: APPROVAL (not APPROVAL_INSTANCE)
- name: "APPROVAL"
  business_purpose: "Tracks approval requests and decisions"
```

Choose one terminology for associated participants and apply it consistently across all entities (e.g., always PARTICIPANT or always RECIPIENT, never mix).

## Appian User References

Never create entities to represent Appian login users (USER, CONSULTANT, AGENT, EMPLOYEE, STAFF). Instead, use USERNAME fields that reference Appian's built-in SYSTEM_USERS record type.

```yaml
# Correct: USERNAME field referencing SYSTEM_USERS
attributes:
  - name: "assigned_username"
    type: "USERNAME"
    required: false
    foreign_key: "SYSTEM_USERS.username"
relationships:
  - name: "assignedUser"
    type: "many-to-one"
    entity: "SYSTEM_USERS"
    foreign_key: "assigned_username"
    references: "username"

# Wrong: Creating a user entity
# - name: "AGENT"
#   attributes:
#     - name: "username" ...
```

SYSTEM_USERS provides: username (PK), firstName, lastName, email, phone fields, address fields, title, supervisor, active status, timezone, locale, and 10 custom fields.

## Relationship Field Requirements

Every relationship declaration must use these exact field names:

| Required Field | Correct Name | Wrong Names |
|---|---|---|
| Relationship identifier | `name` | `relationship_name`, `label` |
| Relationship type | `type` | `relationship_type`, `rel_type` |
| Target table | `entity` | `related_entity`, `target_entity` |
| Foreign key field | `foreign_key` | `fk`, `foreignKey` |
| Referenced field | `references` | `reference`, `referenced_field` |

Valid relationship type values: `"one-to-many"`, `"many-to-one"`, `"one-to-one"`

Every FK attribute must have a corresponding relationship declaration, and every many-to-one or one-to-one relationship must have a corresponding FK attribute in the same table.
