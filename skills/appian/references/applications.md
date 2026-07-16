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

## Reverse Workflow: Finding an Application for an Existing Object

**When:** You have a design object uuid (expression rule, interface, constant, record type) but don't know which application it belongs to.

**Call `getObjectDependents(objectUuid)`**:
   - Returns the owning application UUID in the `dependents` array
   - Filter for `type === "APPLICATION"` and extract `uuid`

---

## Confirmation Patterns

This section covers application-specific patterns for when to ask users for confirmation versus when to automatically complete mandatory steps. Always load `references/confirmation-patterns.md` for universal workflows.

### Application Deletion

**When to use:** Before deleting an application.

**Risk level:** CRITICAL (requires typed confirmation)

**Cross-reference:** `confirmation-patterns.md` → Universal Workflow 1 (Delete Confirmation)

#### What Application Delete Actually Does

**IMPORTANT:** `deleteApplication` removes the **application container only**. It does NOT delete the design objects contained within the application.

**What gets deleted:**
- Application metadata (name, description, prefix, urlIdentifier)
- Application container definition

**What gets PRESERVED:**
- All record types
- All interfaces
- All expression rules
- All process models
- All constants
- All sites
- Auto-generated groups (Administrators, Users)
- Auto-generated folders (Rules and Constants, Artifacts, etc.)

**After deletion:**
- Design objects become **unassociated** (no parent application)
- Objects remain functional but lose application context
- Auto-generated groups and folders may become orphaned
- Prefix conventions no longer enforced

#### Dependency Checks

Before deleting an application, check what objects it contains:

1. **List contained objects** via `listApplicationObjects(uuid)`
   - Returns counts and lists of: record types, interfaces, expression rules, process models, constants, sites, etc.
   
2. **Present findings to user** with clear warning about preservation:

```
You are about to DELETE application "[Name]" (UUID: [uuid]).

This will permanently remove:
- Application container (metadata only)

This will be PRESERVED but unassociated:
- [N] record types
- [N] interfaces
- [N] expression rules
- [N] process models
- [N] constants
- [N] sites
- Auto-generated groups: [GroupNames]
- Auto-generated folders: [FolderNames]

Impact:
- Design objects lose application context
- Objects remain functional but become harder to manage
- Auto-generated groups/folders may become orphaned
- Cannot recreate application with same name immediately

Recommendation: If you want to remove all objects, delete them individually first, then delete the application container.

This action CANNOT be undone.

To confirm, type: DELETE [Name]
```

### Name and Prefix Collision

**When to use:** Before creating an application.

**Cross-reference:** `confirmation-patterns.md` → Universal Workflow 2 (Name Collision Detection)

#### Application Name Collision

Applications are **global objects** (not scoped to other applications). Check across the entire environment.

**Mandatory Step 1:** Call `listApplications()` BEFORE `createApplication`

**Check for two types of collision:**

1. **Name collision:**
   - Exact match: "Case Management" vs "Case Management"
   - Case-insensitive match: "Case Management" vs "case management"
   - Similar names: "Case Management" vs "Case Management System"

2. **Prefix collision:**
   - Exact match: prefix "CM" vs prefix "CM"
   - Case-insensitive match: prefix "CM" vs prefix "cm"
   - Similar prefixes: prefix "CM" vs prefix "CMS" (if same domain)

**Present to user if collision detected:**
```
Found existing application with similar name and/or prefix:

Existing: "[ExistingName]" (prefix: "[ExistingPrefix]")
Proposed: "[ProposedName]" (prefix: "[ProposedPrefix]")

[If name collision] Applications must have unique names.
[If prefix collision] Multiple applications with the same prefix can cause naming conflicts in constants, expression rules, and interfaces.

Would you like to:
1. Use the existing application
2. Create a new application with a different name/prefix
3. Proceed anyway (creates separate application, prefix conflicts possible)

What would you like to do? (1/2/3)
```

**Prefix collision example:**
```
Found application "Case Management" with prefix "CM".

Creating "Customer Management" with prefix "CM" will cause prefix conflicts:
- Constants: CM_ADMIN_GROUP (which app?)
- Expression Rules: CM_GetCaseList vs CM_GetCustomerList (confusing)
- Interfaces: CM_CaseForm vs CM_CustomerForm (ambiguous)

Suggest using prefix "CUST" or "CUSTMGMT" for "Customer Management".

Proceed with prefix "CM" anyway? (yes/no)
```

### Auto-Generated Objects Discovery

**When to use:** Immediately after creating an application.

**Pattern type:** PROACTIVE COMPLETION (automatic, inform user)

**Rule:** After creating an application, automatically call `getApplication(uuid)` to retrieve `defaultObjects` UUIDs.

**Why this matters:**
- Applications create groups and folders automatically
- UUIDs are needed for subsequent operations (security, folder organization)
- Discovering UUIDs prevents having to list/search later

**Workflow:**
1. Call `createApplication` → get application UUID
2. Automatically call `getApplication(uuid)` (no asking)
3. Extract `defaultObjects` from response:
   - `administratorsGroupUuid`
   - `usersGroupUuid`
   - `rulesAndConstantsFolderUuid`
   - `knowledgeCenterUuid`
   - `artifactsFolderUuid`
   - `artifactsFolderConstantUuid`
   - `documentationFolderUuid`
   - `processModelFolderUuid`
4. Report to user with stored UUIDs:

```
✅ Created application "[Name]" (prefix: [PREFIX])

Auto-generated objects:
- [PREFIX] Administrators group (UUID: [uuid])
- [PREFIX] Users group (UUID: [uuid])
- Rules and Constants folder (UUID: [uuid])
- Knowledge Center (UUID: [uuid])
- Artifacts folder (UUID: [uuid])
- Documentation folder (UUID: [uuid])
- Process Model folder (UUID: [uuid])

UUIDs stored for subsequent operations.
```

**Do NOT ask:** "Should I get the application details?" — this is automatic discovery, not optional.

**Example from validated test:**
```
Created "Project Management System" application (PMS prefix)

✅ Auto-discovered default objects:
- PMS Administrators: _e-0000eff3-b4fc-8000-9b00-01075c01075c_181
- PMS Users: _e-0000eff3-b4fc-8000-9b00-01075c01075c_183
- Rules and Constants folder: _a-0000eff3-b56a-8000-9bb3-011c48011c48_9449
[... full list of UUIDs ...]

Ready to create role groups and constants.
```

---

## When You Need More

**Load these references based on your next steps:**
- Group hierarchy and security → `security-patterns.md`
- Record types and data model → `data-modeling.md` + `record-types.md`
- Interfaces and SAIL → `interfaces.md` + `sail.md`
- Process models → `process-models.md` + `node-types.md`
