---
name: "appian-change-planning"
description: "Plan changes to Appian applications — discover what exists, determine scope, identify which objects need to change, order dependencies, and manage the application inventory. Use when the agent receives a development task and needs to figure out what to do."
---

## Environment Discovery

Before creating or modifying anything, discover what already exists. Scope discovery to what the current task needs — don't exhaustively catalog the entire environment.

### What to Discover

- **Applications** — list applications to find the target application and its `appUuid`. If the task references an existing application by name, search for it. If no application exists and the task requires one, plan to create it.
- **Objects within the application** — list record types, interfaces, expression rules, process models, sites, web APIs, constants, groups, and folders scoped to the application's `appUuid`. Only list the object types relevant to the current task.
- **Existing configuration** — when modifying an existing object, retrieve its current state before planning changes. For record types, get the source configuration to see existing fields and the data source details (`dataSourceUuid`, `schema`). For interfaces, list inputs and get the current SAIL expression. For process models, get current nodes and variables.
- **Reusable configuration** — when creating new record types, discover `dataSourceUuid` and `schema` from an existing record type in the same application rather than guessing or hardcoding these values.
- **Folders** — discover existing folders (rule folders, document folders, process model folders) before creating new ones. Applications auto-generate default folders at creation time.
- **Groups** — discover existing groups before creating new ones. Applications auto-generate Administrators and Users groups at creation time.

### Discovery Scope Rules

- **Single-field fix**: retrieve the one object being modified — no broader discovery needed
- **Adding a field to a record type**: retrieve the record type and its source configuration
- **Adding a new record type**: list existing record types (to discover data source config and avoid duplicates), list groups (for security), list folders (for organization)
- **New interface referencing record types**: list record types (to get field UUIDs for data binding), list expression rules (to find reusable logic), list existing interfaces (to avoid duplicates)
- **Full application build**: list everything in the application to understand the current state before planning additions

## Scope Determination

Match the plan's scope to the request. Don't over-plan a small change; don't under-plan a large one.

### Sizing the Plan

- **Single-object change** (add a field, fix a label, update an expression): plan the one change directly. No dependency ordering needed. Discover the object, make the change.
- **Multi-object change** (add a record type with a form and process model): plan the objects in dependency order. Identify which objects need to be created and which already exist.
- **Full application** (greenfield build from requirements): plan all layers — data model, security, interfaces, workflows, site. Use the full dependency ordering below.

### What a Plan Should Include

1. **Objects to create** — listed in dependency order, with enough detail to execute (names, key configuration)
2. **Objects to modify** — what changes and why
3. **Objects that already exist** — confirmed via discovery, no action needed
4. **Manual steps** — anything the tooling can't automate (see untooled object types below)
5. **Dependencies between steps** — which steps must complete before others can start

### What a Plan Should NOT Include

- Exhaustive discovery of objects unrelated to the task
- Speculative objects "in case we need them later"
- Detailed SAIL expressions or process model node configurations — those belong in execution, not planning
- Duplicating guidance that lives in domain skills — the plan identifies *what* to build; domain skills guide *how*

## Dependency Ordering

When a change involves multiple object types, create them in this order so that references resolve correctly. A single-object change skips this entirely. A multi-object change uses only the relevant subset.

1. **Application** — creates default groups (Administrators, Users) and default folders (rules, documents, process model)
2. **Groups** — additional role-based groups beyond the auto-generated pair, if the task requires role-specific security
3. **Folders** — additional sub-folders beyond the auto-generated defaults, if the task requires document organization or rule organization
4. **Constants** — group references, configuration values, status labels; must exist before objects that reference them via `cons!`
5. **Record Types** — with fields and source configuration; create all record types before adding cross-type relationships
6. **Record Type Relationships** — after all record types exist, add relationships between them; both sides of each relationship must be declared
7. **Expression Rules** — may reference record types via `recordType!` and constants via `cons!`
8. **Interfaces** — may reference expression rules via `rule!`, record types, and constants; add inputs before setting the SAIL expression
9. **Process Models** — may reference interfaces (start forms, User Input Tasks), expression rules, record types (Write Records nodes), and constants
10. **Web APIs** — expression bodies may reference record types, expression rules, and constants
11. **Sites** — reference interfaces as page targets; page visibility expressions may reference group constants
12. **Documents** — uploaded to document folders; rarely referenced by other design objects during creation

### Key Ordering Rules

- All record types must exist before any relationships are added between them
- Interface inputs must be added before the SAIL expression that references them
- Groups and constants must exist before visibility expressions or security configurations that reference them
- Process model folders must exist before process models can be created in them
- The interface must exist before a process model references it as a start form or User Input Task

## Application Inventory Management

Every object created for an application should be associated with that application.

- **Pass `appUuid` on creation** — when creating any object (record type, interface, expression rule, process model, constant, group, folder, document, site, web API), include the application's `appUuid`. This automatically adds the object to the application's inventory and applies the application's default security.
- **Add pre-existing objects** — if an object was created without `appUuid` or needs to be associated with a different application, use the tool that adds objects to an application by providing the object's UUID and type.
- **Verify inventory** — after creating objects, you can list objects scoped to the application to confirm they appear in the inventory.

### Object Type Identifiers

When adding objects to an application inventory, each object needs a type identifier:

| Object | Type Identifier |
|---|---|
| Constant | `CONSTANT` |
| Interface | `INTERFACE` |
| Process Model | `PROCESS_MODEL` |
| Expression Rule | `EXPRESSION_RULE` |
| Record Type | `RECORD_TYPE` |
| Site | `SITE` |
| Group | `GROUP` |
| Data Type | `DATA_TYPE` |

## Adversarial Self-Review

After producing a plan, review it before presenting it to the developer. Scale the review to the change size.

### Quick Sanity Check (single-object or small changes)

- Does the plan modify the right object?
- Are there any prerequisites missing (e.g., modifying a field on a record type that doesn't exist yet)?
- Will the change break any existing references?

### Standard Review (multi-object changes)

- Are objects listed in correct dependency order?
- Are all required objects included? (e.g., if creating a form, is there a process model to use it? If creating a site page, does the interface exist?)
- Are there any circular dependencies?
- Do naming conventions follow the application's prefix pattern?
- Are both sides of every relationship declared?
- Are group constants created for every group that will be referenced in expressions?

### Layered Review (full application builds)

Perform the standard review, then additionally:

- **Data model review** — are entities properly normalized? Are junction tables included for many-to-many relationships? Do all entities have primary keys and appropriate audit fields?
- **Security review** — are the right groups defined for the roles in the requirements? Are visibility expressions planned for admin-only pages?
- **Interface review** — is there a dashboard, summary view, and form for each record type that needs user interaction? Do forms handle both create and update modes?
- **Workflow review** — does every form have a backing process model? Do process models handle cancel flows?
- **Site review** — are all user-facing interfaces included as site pages? Is page ordering logical (dashboard first, admin last)?
- **Completeness check** — walk through the user's requirements point by point and confirm each one maps to at least one planned object

## Handling Untooled Object Types

The current tool surface covers a subset of Appian's full design object model. When a task requires object types that can't be automated, include them in the plan and mark them as manual steps.

### Full Appian Object Dependency Graph

This is the complete dependency graph including both tooled and untooled object types. Untooled types are marked with **(manual)**.

1. Application
2. Groups
3. Folders
4. Constants
5. Connected Systems **(manual)** — external system connection configuration (REST, database, etc.)
6. Integrations **(manual)** — specific API calls built on connected systems
7. Data Stores **(manual)** — implicit in record type source configuration for database-backed types, but standalone data stores for non-record-type use cases are manual
8. Record Types (with fields and source configuration)
9. Record Type Relationships
10. AI Skills **(manual)** — document classification, email classification, document extraction; can be referenced as process model nodes but the skill itself must be configured manually
11. Decision Objects **(manual)** — business rules and decision tables
12. Expression Rules
13. Interfaces
14. Record Views **(manual)** — configured on record types but not directly toolable; summary views are built as interfaces and linked manually
15. Record Actions **(manual)** — process models can serve as record actions, but the record action configuration linking a process model to a record type is manual
16. Process Models
17. Web APIs
18. Portals **(manual)** — public-facing web applications; separate from sites
19. Sites
20. Documents

### How to Handle Manual Steps

When the plan includes untooled object types:

1. **Include them in the plan** at the correct position in the dependency order
2. **Mark them clearly as manual** — the developer needs to know which steps require their intervention
3. **Continue with automatable steps** — don't block the entire plan because one step is manual
4. **Document what the manual step needs** — provide enough detail (name, purpose, configuration requirements) for the developer to complete it
5. **Note downstream dependencies** — if an automatable object depends on a manual one (e.g., a process model node that calls an integration), flag that the automated step may need adjustment after the manual step is complete

