---
name: "appian-data-modeling"
description: "Design relational data models for Appian applications — entity design, normalization, relationships, junction tables, and reference tables. Covers naming conventions, field types, primary keys, audit fields, and schema validation. Use when designing schemas, planning data structures, classifying entities vs. reference tables, or working with existing databases."
---

## Designing Entities

Every transactional entity needs a surrogate INTEGER primary key — never use composite keys. Appian record types require single-column primary keys for platform compatibility.

### Entity Template

Follow this structure for every entity:

1. Surrogate primary key: `[table_name_lowercase]_id` (INTEGER, auto-increment)
2. Business fields from requirements
3. Foreign keys to reference tables and related entities
4. Audit fields (always include on entities, never on reference or junction tables)

### Audit Fields

Add these four fields to every entity:

- `created_by_username` (USERNAME) — who created the record, references SYSTEM_USERS
- `modified_by_username` (USERNAME) — who last modified the record, references SYSTEM_USERS
- `created_at` (DATETIME) — when the record was created
- `updated_at` (DATETIME) — when the record was last modified

Do not add audit fields to reference tables or junction tables. Reference tables are static lookup data maintained by admins; junction tables contain only foreign keys.

### Naming Conventions

Follow these naming rules consistently:

- **Table names**: UPPER_SNAKE_CASE, singular nouns (CASE, PROVIDER, LETTER_OF_AUTHORIZATION)
- **Column names**: snake_case for all database columns (Appian converts to camelCase in record types)
- **Primary keys**: `[table_name_lowercase]_id` (e.g., table CASE → `case_id`)
- **Foreign keys**: `[referenced_table_lowercase]_id` (e.g., referencing CASE_STATUS → `case_status_id`)
- **Status fields**: `[context]_status`
- **Date fields**: `[context]_date`; use `_at` suffix for DATETIME, `_on` suffix for DATE
- **Boolean fields**: `is_[condition]`
- **No table prefixes** on column names

### Table Name Rules

- Use full, unambiguous names — no abbreviations except approved ones (NPI, TIN, NDA, ZIP, URL, US)
- Forbidden abbreviations: AUTH (use AUTHORIZATION), LOA (use LETTER_OF_AUTHORIZATION), COMM (use COMMUNICATION), INFO (use INFORMATION), REQ (use REQUEST)
- Keep table names under 30 characters for Oracle compatibility. If over 30, first remove context words (NETWORK, SYSTEM, RECORD, DATA), then use approved abbreviations only as a last resort
- Status tables must follow `[ENTITY]_STATUS` format — never use generic STATUS alone
- Use entity-scoped names unless a reference table is shared by 3+ entities from different functional areas

### Column Name Constraints

- Use snake_case consistently — Appian auto-converts to camelCase for record type fields
- Avoid SQL reserved keywords as column names
- Keep column names under 30 characters for Oracle compatibility

## Normalization Principles

Normalize to at least Third Normal Form (3NF) for all transactional tables. Appian's data layer works best with properly normalized schemas.

### When to Normalize

- Enumerated values with 3+ options → extract to a reference table (always, no exceptions)
- Multi-valued attributes → extract to a junction table (4NF)
- Repeating groups of fields → extract to a child entity with a foreign key back to the parent
- Fields that depend on non-key attributes → separate into their own table

### When to Denormalize

- Binary values (exactly 2 universal options like Yes/No, True/False) → keep as `allowed_values` on the field or use BOOLEAN
- One-to-one data with 3 or fewer fields (excluding PK/FK) → keep inline on the parent entity
- Requirements explicitly say data is "stored with" or "stored on" a parent entity → keep as fields on that entity

### The 1:1 Decision Tree

When you encounter a one-to-one relationship:

- **1-3 fields**: keep inline on the parent entity
- **4+ fields**: consider a separate entity (especially for sensitive data separation, optional attribute groups, or performance optimization)
- **Separate entity requires**: FK field with `unique: true` to enforce the 1:1 constraint, and bidirectional one-to-one relationships declared on both sides

## Classifying Tables

Every table in the model falls into one of three categories: entity, reference table, or junction table. Correct classification drives the entire downstream model.

### Reference Table Detection

Apply the 5-criteria test to determine if a table is a reference table:

1. **Small, static dataset** (< 50 records) — stores configuration/lookup data, not transaction records
2. **Controlled vocabulary** — simple structure: id + code/name + enabled/active (may include admin audit fields)
3. **Lookup/validation purpose** — populates dropdowns, validates selections, filters data
4. **No temporal/transactional attributes** — no workflow tracking fields (status changes, due dates, assignments). Admin audit fields like created_by/modified_by are fine.
5. **Minimal relationships** — only one-to-many outward (other entities point to it). May have FK to a parent reference table (hierarchical lookups).

If a table passes all 5 criteria → reference table. If it fails any one → entity.

If requirements explicitly classify a table ("Reference table of...", "Lookup table for..."), follow the explicit classification and skip the 5-criteria test.

### Entity vs. Reference Table Quick Checks

- Does it grow with user actions? → Entity
- Does it have workflow/lifecycle fields? → Entity
- Does it have FKs to entity tables? → Entity
- Is it a small, static list that populates dropdowns? → Reference table
- Does its name end in _TYPE, _STATUS, _CATEGORY, _PRIORITY? → Likely reference table (confirm with 5-criteria test)

### Junction Table Identification

Create junction tables when:
- Requirements describe many-to-many relationships between two entities
- Multi-select fields need normalization (4NF)
- The relationship itself has attributes (temporal, audit, or metadata)

## Relationship Patterns

All relationships must be bidirectional. Appian record types require each side to declare its relationships for proper traversal.

### One-to-Many

The most common pattern. The "many" side holds the foreign key.

- **Many side**: declares a `many-to-one` relationship + has the FK attribute
- **One side**: declares a `one-to-many` relationship (no FK attribute needed)
- FK attribute and relationship must both exist on the many side — never add one without the other

### Many-to-Many via Junction Tables

Implement many-to-many relationships through a junction table with:

- Surrogate INTEGER primary key: `[entity1]_[entity2]_id`
- Two FK attributes referencing the connected entities
- Two `many-to-one` relationships (one to each connected entity)
- Both connected entities declare `one-to-many` to the junction table

Junction tables enforce uniqueness on the FK pair at the database level to prevent duplicate relationships.

Add additional attributes to junction tables only when requirements explicitly mention temporal data (date_assigned, effective_date), audit data (created_by), or relationship metadata (is_primary, sort_order). Default to only the two FKs.

### Self-Referential Relationships

For hierarchies (parent/child, manager/subordinate, category trees):

- FK field: `parent_[entity_name]_id` with `required: false` (allows root records)
- Declare both `many-to-one` (child → parent) and `one-to-many` (parent → children) in the same entity
- Only add self-referential relationships when requirements explicitly mention hierarchy, or when the entity type is inherently hierarchical (CATEGORY, DEPARTMENT)
- Never speculatively add self-referential relationships — they add significant query and UI complexity

### One-to-One Relationships

Use when separating sensitive data, optional attribute groups, or when a 1:1 related entity has 4+ fields:

- The side with the FK declares `one-to-one` and has the FK attribute with `unique: true`
- The other side declares `one-to-one` without a FK attribute
- Both sides must declare the relationship for bidirectional access

### Relationship Naming

Relationship names are algorithmically derived for consistency:

- **Many-to-one**: remove `_id` from FK field, convert to camelCase (`customer_id` → `customer`, `case_status_id` → `caseStatus`)
- **One-to-many to entities**: pluralize target entity name in camelCase (`CASE_NOTE` → `caseNotes`)
- **One-to-many to junction tables**: pluralize the other connected entity in camelCase (CASE → CASE_TAG → TAG: name is `tags`)
- **Self-referential inverse**: use semantic plural (`parent_case_id` inverse → `childCases`)

## Reference Table Design

Reference tables store controlled vocabularies — status codes, priority levels, category lists, type enumerations.

### Standard Reference Table Structure

Every reference table follows this template:

- `[table_name_lowercase]_id` (INTEGER, PK, auto-increment)
- `[table_name_lowercase]_code` (TEXT, unique, max 20 chars) — short code for storage and lookups
- `[table_name_lowercase]_name` (TEXT, max 100 chars) — display name for UI
- `known_values` — the enumerated values this table contains

### Source Classification

- **Requirements**: only when requirements explicitly say "create a table" or "maintain a table"
- **Best Practice**: when normalizing a controlled value field (dropdown with 3+ values) into a reference table — this is the most common case
- **Suggested**: when standardizing a free-text field into a controlled vocabulary

The field mentioned in requirements may have source="Requirements", but the reference table itself is typically source="Best Practice" because creating the table is a design decision.

### Binary Values (2 Options)

For universal binary concepts (Yes/No, True/False, Active/Inactive), use `allowed_values` on the field or BOOLEAN type — do not create a reference table. For domain-specific pairs (Approved/Rejected, Form/Request), create a reference table even though there are only 2 values.

## Schema Validation

Before finalizing a data model, verify these rules:

### Structural Checks
- Every table has a single-column INTEGER primary key (no composite keys)
- All junction tables have exactly 2 FK attributes and 2 many-to-one relationships
- All FK attributes have a corresponding many-to-one or one-to-one relationship declared
- All one-to-many relationships have a corresponding many-to-one on the other side
- No orphaned tables without relationships

### Naming Checks
- Table names are UPPER_SNAKE_CASE and singular
- Column names are snake_case
- Relationship names are camelCase with no underscores
- PK fields match `[table_name_lowercase]_id` pattern
- FK fields match `[referenced_table_lowercase]_id` pattern
- Status tables follow `[ENTITY]_STATUS` format

### Consistency Checks
- No duplicate attribute names within any table
- All relationship names within a table are unique
- Every USERNAME field has `foreign_key: "SYSTEM_USERS.username"` and a corresponding many-to-one relationship to SYSTEM_USERS
- Reference tables have no audit fields (created_at, updated_at, etc.)
- Junction tables have only PK + FK fields (plus explicitly required metadata)

### Platform Compatibility
- Only valid Appian field types used: TEXT, INTEGER, DECIMAL, DATE, DATETIME, BOOLEAN, USERNAME, GROUP, DOCUMENT
- No user/role entities created — use USERNAME fields referencing SYSTEM_USERS instead
- All tables conform to at least 3NF; transactional tables conform to 4NF

## Document Handling

- For entities requiring exactly one document per record: add a DOCUMENT field directly to the entity
- For entities requiring multiple documents per record: create a separate `[ENTITY]_DOCUMENT` table with a DOCUMENT field and a one-to-many relationship from the parent entity
- Use entity-specific naming for document tables (VEHICLE_DOCUMENT, MAINTENANCE_DOCUMENT)

## Pitfalls

- Using composite primary keys — Appian record types require single-column INTEGER surrogate keys
- Creating user/role entities (USER, AGENT, EMPLOYEE) — use USERNAME fields referencing Appian's built-in SYSTEM_USERS instead
- Adding audit fields to reference tables or junction tables — only entities get audit fields
- Using generic STATUS table — always scope to the entity: CASE_STATUS, ORDER_STATUS
- Using abbreviated table names (AUTH, LOA, COMM) — use full names for clarity and cross-scope consistency
- Forgetting bidirectional relationships — both sides must declare the relationship for Appian record type traversal
- Adding a relationship without the corresponding FK attribute (or vice versa) — both must exist together
- Speculatively adding self-referential relationships — only add when requirements explicitly mention hierarchy
- Using `allowed_values` for 3+ options — always create a reference table for 3+ enumerated values
- Creating junction tables when a simple FK suffices — "assign case to worker" (singular) needs a FK, not a junction table; "assign multiple workers to case" needs a junction table

## When You Need More

For questions about specific Appian data types, advanced schema patterns, or database configuration beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.

Load `references/entity-patterns.md` when you need entity design templates and field conventions.

Load `references/relationship-patterns.md` when you need relationship type details, junction table patterns, and reference table patterns.
