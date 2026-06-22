# Applications

## Create JSON Schema

```json
{
  "name": "Case Management",
  "prefix": "CM",
  "description": "Optional description"
}
```

- `name` (required) — application display name
- `prefix` (optional) — naming prefix for objects (e.g., "CM"). Auto-generated from name if omitted.
- `description` (optional)

## What Creation Produces

Creating an application auto-generates:
- **PREFIX Administrators** group — developers and admins
- **PREFIX Users** group — all end users (includes Administrators as members)
- Default rule folder, document folder (Knowledge Center), and process model folder

Capture the application UUID from the create response for use in subsequent operations.

## Discovery After Creation

After creating an app, discover its auto-generated objects by getting the application details. Look for the `defaultObjects` field which contains UUIDs for the auto-generated groups, folders, and process model folder.

## Application Prefix Convention

The prefix propagates to all objects in the application:
- Record types: `CM Case`, `CM Customer` (prefix + Title Case singular; plural name has no prefix)
- Interfaces: `CM_Dashboard`, `CM_CaseForm`
- Expression rules: `CM_GetFullName`, `CM_IsEligible`
- Process models: `CM Create Case`, `CM Reassign Case`
- Constants: `CM_ADMIN_GROUP`, `CM_STATUS_OPEN`
- Groups: `CM Administrators`, `CM Case Managers`
- Sites: `CM Case Management` (object name), `Case Management` (display name)

---

## Group Hierarchy for Multi-Role Applications

**If your application has multiple user roles** (Managers, Workers, Submitters, Reviewers), you MUST set up a group hierarchy after creating the application.

**Auto-generated groups (created with application):**
- `PREFIX Administrators` — developers and admins
- `PREFIX Users` — all end users

**For role-based applications, add role groups with parent-child relationships:**
```
PREFIX Users (auto-generated)
├── PREFIX Administrators (auto-generated, member of Users)
├── PREFIX Managers (custom, member of Users)
├── PREFIX Case Workers (custom, member of Users)
└── PREFIX Submitters (custom, member of Users)
```

**Critical:** Use `parentGroupName` parameter when creating role groups to establish hierarchy. This enables permission inheritance — users in child groups inherit permissions from `PREFIX Users`.

**After creating role groups, create group constants:**
- `PREFIX_ADMIN_GROUP` → PREFIX Administrators
- `PREFIX_ALL_USERS_GROUP` → PREFIX Users  
- `PREFIX_ROLE1_GROUP` → PREFIX [First Role] (e.g., Managers, Administrators, Staff)
- `PREFIX_ROLE2_GROUP` → PREFIX [Second Role] (e.g., Workers, Processors, Supervisors)

These constants are required for security expressions in interfaces, sites, record actions, and record-level security. Use group-specific names (e.g., MANAGER_GROUP, CASE_WORKER_GROUP, PROCESSOR_GROUP) that match your application's roles.

**See `security-patterns.md` for:**
- Complete group hierarchy templates
- Group constant naming patterns
- Security expression examples
- Role-based access patterns

---

## Common Failure Modes

**❌ Creating role groups without parent-child relationships:**
- Symptom: All groups at same level, no permission inheritance, manual permission management on every object
- Cause: Didn't load `security-patterns.md` before creating groups
- Example: Created "CM Managers", "CM Case Workers", "CM Submitters" but didn't set `parentGroupName` to "CM Users"
- Pattern: Applies to ANY role-based application (Managers/Workers/Reviewers, Administrators/Processors/Auditors, Staff/Supervisors/Directors, etc.)
- Prevention: Load `security-patterns.md` → follow group hierarchy template → use `parentGroupName` parameter

**❌ Missing group constants:**
- Symptom: Can't write security expressions, hard-coded group names in expressions break when groups renamed
- Cause: Created groups but didn't create constants for them
- Example: Security expression references `cons!CM_MANAGER_GROUP` but constant doesn't exist → runtime error
- Prevention: Load `security-patterns.md` → see group constants pattern → create constant for every role group

**❌ Wrong data model conventions:**
- Symptom: Record types have `caseId` instead of `id` for primary key, TEXT fields instead of reference tables for status/priority
- Cause: Didn't load `data-modeling.md` before creating record types
- Prevention: Load `data-modeling.md` before creating any record types in the application

**❌ Missing USER field relationships:**
- Symptom: USER fields (assignedTo, createdBy) display as plain text, no navigation to user profiles
- Cause: Created USER fields but didn't add relationships to SYSTEM_RECORD_TYPE_USER
- Prevention: Load `record-types.md` + `relationship-patterns.md` → follow USER field relationship pattern

---

## Complete Application Creation Workflow

**Correct sequence** for multi-role applications (with proper skill loading):

1. **Load skills FIRST:**
   - `SKILL.md` → dependency ordering
   - `applications.md` (this file) → creation syntax
   - `security-patterns.md` → group hierarchy
   - `data-modeling.md` → naming conventions

2. **Create application:**
   - `createApplication` → capture UUID and defaultObjects UUIDs

3. **Create role groups with hierarchy:**
   - `createGroup` for each role → set `parentGroupName` to "PREFIX Users"
   - Capture each group UUID

4. **Create group constants:**
   - `createConstant` for each group → reference in security expressions

5. **Create data model:**
   - Load `record-types.md` + `relationship-patterns.md`
   - Create reference tables (Status, Priority) → load sample data
   - Create entity tables → add relationships (bidirectional) → load sample data

6. **Create interfaces, process models, sites:**
   - Load appropriate reference files for each object type
   - Use group constants in security expressions

---

## When You Need More

**Load these references based on your next steps:**
- Group hierarchy and security → `security-patterns.md`
- Record types and data model → `data-modeling.md` + `record-types.md`
- Interfaces and SAIL → `interfaces.md` + `sail.md`
- Process models → `process-models.md` + `node-types.md`
