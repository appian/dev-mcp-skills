# Relationship Patterns Reference

## Bidirectional Relationship Rule

All relationships must be bidirectional across all table types (entities, reference tables, junction tables). This is required for Appian record type compatibility — each record type must declare its own relationships for traversal to work.

**Pattern**: The table with the FK attribute is the "many" side and declares `many-to-one`. The referenced table is the "one" side and declares `one-to-many`.

## One-to-Many Relationships

The most common relationship pattern. The child entity holds the foreign key.

### Child Side (Many)

Has both the FK attribute and the many-to-one relationship:

```yaml
# CASE entity (child, many side)
attributes:
  - name: "customer_id"
    type: "INTEGER"
    required: true
    foreign_key: "CUSTOMER.customer_id"
    description: "Reference to customer"
relationships:
  - name: "customer"
    type: "many-to-one"
    entity: "CUSTOMER"
    foreign_key: "customer_id"
    references: "customer_id"
    description: "Each case belongs to one customer"
```

### Parent Side (One)

Has only the one-to-many relationship (no FK attribute):

```yaml
# CUSTOMER entity (parent, one side)
relationships:
  - name: "cases"
    type: "one-to-many"
    entity: "CASE"
    foreign_key: "customer_id"
    references: "customer_id"
    description: "A customer can have multiple cases"
```

## Many-to-Many via Junction Tables

Implement many-to-many relationships through a junction table. Never put multi-valued data directly in a column.

### Junction Table Structure

```yaml
junction_tables:
  - name: "CASE_TAG"
    description: "Many-to-many relationship between cases and tags"
    attributes:
      - name: "case_tag_id"
        type: "INTEGER"
        primary_key: true
        auto_increment: true
        description: "Unique identifier"
      - name: "case_id"
        type: "INTEGER"
        required: true
        foreign_key: "CASE.case_id"
        description: "Reference to case"
      - name: "tag_id"
        type: "INTEGER"
        required: true
        foreign_key: "TAG.tag_id"
        description: "Reference to tag"
    relationships:
      - name: "case"
        type: "many-to-one"
        entity: "CASE"
        foreign_key: "case_id"
        references: "case_id"
        description: "Links to case"
      - name: "tag"
        type: "many-to-one"
        entity: "TAG"
        foreign_key: "tag_id"
        references: "tag_id"
        description: "Links to tag"
```

### Connected Entities

Both connected entities declare one-to-many to the junction table:

```yaml
# CASE entity
relationships:
  - name: "tags"
    type: "one-to-many"
    entity: "CASE_TAG"
    foreign_key: "case_id"
    references: "case_id"
    description: "Tags associated with this case"

# TAG entity
relationships:
  - name: "cases"
    type: "one-to-many"
    entity: "CASE_TAG"
    foreign_key: "tag_id"
    references: "tag_id"
    description: "Cases using this tag"
```

### Junction Table Naming

- Table name: `[ENTITY1]_[ENTITY2]` in UPPER_SNAKE_CASE
- Primary key: `[entity1]_[entity2]_id`
- FK fields: `[entity1]_id` and `[entity2]_id`

### When to Create Junction Tables

Create a junction table when:
- "Select multiple [items] from a list" where items represent a separate entity
- "Assign [entities] to multiple [other entities]"
- Many-to-many relationships between distinct business entities
- The relationship itself has temporal attributes, status, or business rules

Do not create a junction table when:
- Requirements use singular language ("assign case to worker") — use a simple FK instead
- No domain precedent for many-to-many exists
- When in doubt, use a simple FK — it's easier to upgrade to a junction table later

### Additional Attributes on Junction Tables

By default, junction tables have only the surrogate PK and two FKs. Add additional attributes only when requirements explicitly mention:

- **Temporal**: `date_assigned`, `date_removed`, `effective_date` — when the relationship has a lifecycle
- **Audit**: `created_by`, `created_date` — when who/when matters for compliance
- **Metadata**: `is_primary`, `sort_order`, `notes` — when the relationship itself has properties

### Junction Table Uniqueness

At the database level, junction tables enforce `UNIQUE(entity1_id, entity2_id)` to prevent duplicate relationships. This is implicit for all junction tables.

## Self-Referential Relationships

For hierarchies where an entity references itself (parent/child, manager/subordinate, category trees).

### Pattern

Both many-to-one and one-to-many are declared in the same entity:

```yaml
- name: "CASE"
  attributes:
    - name: "parent_case_id"
      type: "INTEGER"
      required: false  # Must be false to allow root records
      foreign_key: "CASE.case_id"
      description: "Reference to parent case"
  relationships:
    - name: "parentCase"
      type: "many-to-one"
      entity: "CASE"
      foreign_key: "parent_case_id"
      references: "case_id"
      description: "Each case may have one parent"
    - name: "childCases"
      type: "one-to-many"
      entity: "CASE"
      foreign_key: "parent_case_id"
      references: "case_id"
      description: "A case can have multiple children"
```

### Rules

- FK field must have `required: false` to allow root records (records with no parent)
- FK field naming: `parent_[entity_name]_id`
- Only add when requirements explicitly mention hierarchy, or when the entity type is inherently hierarchical (CATEGORY, DEPARTMENT)
- Never speculatively suggest self-referential relationships — they add significant complexity

## One-to-One Relationships

Use when separating sensitive data, optional attribute groups, or when 1:1 related data has 4+ fields.

### Side With FK

```yaml
# CASE_CONFIDENTIAL (has the FK, enforces 1:1 with unique constraint)
attributes:
  - name: "case_id"
    type: "INTEGER"
    required: true
    unique: true  # Enforces 1:1
    foreign_key: "CASE.case_id"
    description: "Reference to case"
relationships:
  - name: "case"
    type: "one-to-one"
    entity: "CASE"
    foreign_key: "case_id"
    references: "case_id"
    description: "Linked case record"
```

### Side Without FK

```yaml
# CASE (referenced entity, no FK attribute needed)
relationships:
  - name: "caseConfidential"
    type: "one-to-one"
    entity: "CASE_CONFIDENTIAL"
    foreign_key: "case_id"
    references: "case_id"
    description: "Confidential data for this case"
```

### When to Use 1:1

- **Separate sensitive data**: isolate PII or confidential fields into a restricted table
- **Optional attribute groups**: large set of optional fields that only some records use
- **Performance**: separate rarely-queried fields from frequently-queried ones

### 1:1 Decision Tree

| Fields (excluding PK/FK) | Action |
|---|---|
| 1-3 fields | Keep inline on parent entity |
| 4+ fields | Consider separate entity with 1:1 relationship |

## Reference Table Patterns

Reference tables store controlled vocabularies. They have a standard structure and specific relationship rules.

### Standard Reference Table

```yaml
reference_tables:
  - name: "CASE_STATUS"
    description: "Status values for cases"
    attributes:
      - name: "case_status_id"
        type: "INTEGER"
        primary_key: true
        auto_increment: true
        description: "Unique identifier"
      - name: "case_status_code"
        type: "TEXT"
        required: true
        unique: true
        max_length: 20
        description: "Short code for storage and lookups"
      - name: "case_status_name"
        type: "TEXT"
        required: true
        max_length: 100
        description: "Display name for UI"
    known_values:
      - "New"
      - "In Progress"
      - "Resolved"
      - "Closed"
```

### Reference Table Relationships

Reference tables declare one-to-many relationships to each entity that references them. These are added after all entities are defined, during the scope filtering phase.

```yaml
# CASE_STATUS reference table (after relationships are added)
relationships:
  - name: "cases"
    type: "one-to-many"
    entity: "CASE"
    foreign_key: "case_status_id"
    references: "case_status_id"
    description: "Cases with this status"
```

### Entity-Scoped vs. Shared Reference Tables

| Pattern | When to Use | Example |
|---|---|---|
| Entity-scoped: `[ENTITY]_STATUS` | Default — use for most reference tables | CASE_STATUS, ORDER_STATUS |
| Shared: `PRIORITY` | Only when 3+ entities from different functional areas use the same values | PRIORITY (used by CASE, ORDER, TICKET) |

### Hierarchical Reference Tables

Reference tables can have FK relationships to parent reference tables:

```yaml
# CARRIER_PRODUCT references CARRIER (both are reference tables)
- name: "CARRIER_PRODUCT"
  attributes:
    - name: "carrier_product_id"
      type: "INTEGER"
      primary_key: true
      auto_increment: true
    - name: "carrier_id"
      type: "INTEGER"
      required: true
      foreign_key: "CARRIER.carrier_id"
    - name: "carrier_product_name"
      type: "TEXT"
      required: true
      max_length: 100
```

This is valid — reference tables can have FKs to other reference tables (parent lookups). A reference table with FKs to entity tables is likely misclassified and should be an entity instead.

## Relationship Naming Algorithm

### Many-to-One

Remove `_id` suffix from FK field name, convert to camelCase:

| FK Field | Relationship Name |
|---|---|
| `customer_id` | `customer` |
| `assigned_user_id` | `assignedUser` |
| `parent_case_id` | `parentCase` |
| `case_status_id` | `caseStatus` |

### One-to-Many (to Entities)

Pluralize target entity name, convert to camelCase:

| Target Entity | Relationship Name |
|---|---|
| `CASE` | `cases` |
| `CASE_NOTE` | `caseNotes` |
| `CASE_ATTACHMENT` | `caseAttachments` |
| `CATEGORY` | `categories` |
| `ADDRESS` | `addresses` |

CamelCase conversion: split by underscores → lowercase first word → capitalize first letter of subsequent words → join → pluralize.

Pluralization rules:
- Most nouns: add `s`
- Ends in s, x, z, ch, sh: add `es`
- Ends in consonant + y: change y to `ies`
- Irregular: use standard English (PERSON → `people`, CHILD → `children`)

### One-to-Many (to Junction Tables)

Use the other connected entity's name (pluralized, camelCase):

| Source Entity | Junction Table | Target Entity | Relationship Name |
|---|---|---|---|
| CASE | CASE_TAG | TAG | `tags` |
| TAG | CASE_TAG | CASE | `cases` |
| CASE | CASE_SKILL | SKILL | `skills` |
| ORDER | ORDER_PRODUCT | PRODUCT | `products` |

### One-to-One

- **Side with FK**: same as many-to-one (remove `_id`, camelCase)
- **Side without FK**: target entity name in camelCase, singular (`CASE_CONFIDENTIAL` → `caseConfidential`)

### Self-Referential Inverse

Use semantic plural form:

| FK Field | Forward Name | Inverse Name |
|---|---|---|
| `parent_case_id` | `parentCase` | `childCases` |
| `manager_id` | `manager` | `subordinates` |
| `parent_category_id` | `parentCategory` | `childCategories` |

## FK Transformation During Scope Filtering

When building the final data model, TEXT fields that correspond to included reference tables are transformed to INTEGER FK fields:

- Original field: `case_category` (TEXT)
- Reference table: `CASE_CATEGORY`
- Transformed field: `case_category_id` (INTEGER, FK to CASE_CATEGORY.case_category_id)

The FK field name is derived from the reference table name (lowercase + `_id`), not from the original TEXT field name. This ensures consistent, predictable naming.

If a reference table is excluded from the final scope, the field stays as TEXT instead of being transformed to a FK.

## USER Field Relationships to SYSTEM_RECORD_TYPE_USER

USER fields store references to Appian authenticated users. Every USER field MUST have a MANY_TO_ONE relationship to the platform's system user record type.

### CRITICAL Requirement

**Every USER field requires a MANY_TO_ONE relationship to SYSTEM_RECORD_TYPE_USER.**

Without this relationship:
- Records cannot traverse to user details
- USER fields appear as text usernames only (no name, email, profile picture)
- Related records queries fail
- Record views and forms break
- Relationship traversal fails in interfaces

This is not optional. The relationship must be added for every USER field.

### When to Use USER Fields

**Use USER type for:**
- Appian authenticated users in the system
- Audit fields tracking who performed actions (createdBy, modifiedBy)
- Assignment and ownership fields (assignedTo)
- Approval workflow fields (approvedBy, requestedBy)
- Issue reporting (reportedBy)

**Common USER field names:**
- `createdBy` — User who created the record
- `modifiedBy` — User who last modified the record
- `assignedTo` — User assigned to work on the record
- `approvedBy` — User who approved the record
- `requestedBy` — User who requested the record
- `reportedBy` — User who reported the issue/ticket

**Do NOT use USER type for:**
- ❌ External users (customers, vendors) → Use TEXT field + separate entity
- ❌ Employees who aren't Appian users → Create Employee entity
- ❌ Roles or groups → Use GROUP field type instead

### Required Relationship Parameters

For each USER field, create one MANY_TO_ONE relationship with these exact parameters:

**Parameters:**
- `relationshipName`: `{fieldName}User` (algorithmically derived)
  - `createdBy` → `createdByUser`
  - `modifiedBy` → `modifiedByUser`
  - `assignedTo` → `assignedToUser`
  - `approvedBy` → `approvedByUser`
  - `requestedBy` → `requestedByUser`
  - `reportedBy` → `reportedByUser`
- `sourceRecordTypeFieldUuid`: UUID of the USER field (obtained from field creation response)
- `targetRecordTypeFieldUuid`: `"SYSTEM_RECORD_TYPE_USER_FIELD_username"` (literal string constant)
- `targetRecordTypeUuid`: `"SYSTEM_RECORD_TYPE_USER"` (literal string constant)
- `relationshipType`: `"MANY_TO_ONE"`

**Important Platform Constants:**
- `SYSTEM_RECORD_TYPE_USER` — System record type UUID (constant across all environments)
- `SYSTEM_RECORD_TYPE_USER_FIELD_username` — Username field UUID (constant across all environments)

These are **literal string values**, not variables to retrieve. Use exactly as shown in quotes.

### Field Naming Conventions

**Correct field names:**
- ✅ `createdBy` (not `createdByUsername`, `createdByUser`)
- ✅ `modifiedBy` (not `modifiedByUsername`, `lastModifiedBy`)
- ✅ `assignedTo` (not `assignedToUser`, `assignee`)
- ✅ `approvedBy` (not `approver`, `approvedByUser`)
- ✅ `requestedBy` (not `requestedByUser`)
- ✅ `reportedBy` (not `reportedByUser`)

**Why:** Field name describes the role/action, not the type. The USER type already indicates it's a user reference. Avoid redundant suffixes like `Username` or `User` in field names.

### Relationship Naming Algorithm

Append `User` to the USER field name:

| USER Field Name | Relationship Name |
|---|---|
| createdBy | createdByUser |
| modifiedBy | modifiedByUser |
| assignedTo | assignedToUser |
| approvedBy | approvedByUser |
| requestedBy | requestedByUser |
| reportedBy | reportedByUser |

The relationship name is always `{fieldName}User`, NOT the same as the field name.

### Execution Order

USER field relationships should be added in this sequence:

1. **After FK relationships to reference tables/entities** (statusId, priorityId, categoryId, customerId)
2. **Before entity sample data insertion** (sample data needs relationships in place for traversal)
3. **Sequentially** (one at a time, not in parallel, to avoid version conflicts)

**Example order for Ticket entity with 3 USER fields:**
```
Step 1: Create Ticket record type with fields:
  - id (INTEGER, PK)
  - title (TEXT)
  - statusId (INTEGER, FK)
  - priorityId (INTEGER, FK)
  - createdBy (USER)
  - modifiedBy (USER)
  - assignedTo (USER)

Step 2: Add FK relationships to reference tables (bidirectional):
  - Ticket.statusId → Status.id (MANY_TO_ONE)
  - Status.id → Ticket.statusId (ONE_TO_MANY)
  - Ticket.priorityId → Priority.id (MANY_TO_ONE)
  - Priority.id → Ticket.priorityId (ONE_TO_MANY)

Step 3: Add USER field relationships (sequential, one at a time):
  - Ticket.createdBy → SYSTEM_RECORD_TYPE_USER (MANY_TO_ONE, relationshipName: "createdByUser")
  - Ticket.modifiedBy → SYSTEM_RECORD_TYPE_USER (MANY_TO_ONE, relationshipName: "modifiedByUser")
  - Ticket.assignedTo → SYSTEM_RECORD_TYPE_USER (MANY_TO_ONE, relationshipName: "assignedToUser")

Step 4: Insert entity sample data (all relationships now exist)

Step 5: Set title expression
```

**Key points:**
- Add USER relationships sequentially, not in parallel
- Each relationship addition increments the record type's versionId
- Use the updated versionId from each response for the next relationship
- USER field relationships come after FK relationships but before sample data

### Sample Data with USER Fields

Before generating sample data with USER field values:

1. Query SYSTEM_RECORD_TYPE_USER to retrieve real usernames from the target environment
2. Use actual usernames from the query results
3. **Never fabricate usernames** like "user1", "admin", "john.doe" — they may not exist

**Why this matters:** Fabricated usernames cause sample data insertion to fail or create broken references. USER fields must contain valid usernames that exist in the Appian environment.

**Example valid sample data (after querying for real usernames):**
```
id,title,statusId,priorityId,createdBy,modifiedBy,assignedTo
1,Login issue,1,1,jupiter.munoz,jupiter.munoz,jupiter.munoz
2,Reset password request,2,2,jupiter.munoz,jupiter.munoz,jupiter.munoz
3,Cannot access reports,1,1,jupiter.munoz,jupiter.munoz,jupiter.munoz
```

### Common Mistakes

- ❌ **Forgetting USER field relationships** — Adding USER fields but omitting the required relationships to SYSTEM_RECORD_TYPE_USER
- ❌ **Wrong target UUIDs** — Using variables instead of literal string constants
- ❌ **Wrong relationship name** — Using `createdBy` (field name) instead of `createdByUser` (relationship name)
- ❌ **Parallel relationship addition** — Adding multiple USER relationships simultaneously causes version conflicts
- ❌ **Creating user entities** — Creating a custom USER or EMPLOYEE table instead of using USER fields with system relationships
- ❌ **Wrong field naming** — Using `createdByUsername` instead of `createdBy`
- ❌ **Fabricating usernames in sample data** — Using made-up usernames that don't exist in the environment
- ❌ **Skipping relationships** — Assuming USER fields work without explicit relationships

### Verification

After adding USER field relationships, verify by listing the record type's relationships.

**Expected relationships for entity with 2 USER fields (createdBy, modifiedBy):**
- `createdByUser` (MANY_TO_ONE to SYSTEM_RECORD_TYPE_USER) ✅
- `modifiedByUser` (MANY_TO_ONE to SYSTEM_RECORD_TYPE_USER) ✅

If missing: Add the missing USER field relationships before proceeding to sample data insertion.
