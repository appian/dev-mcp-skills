# Security Patterns Reference

## Standard Group Hierarchy Template

When creating an Appian application, the platform auto-generates two groups. Extend this hierarchy based on the application's role requirements.

### Auto-Generated Groups (created with the application)

| Group | Naming Pattern | Members | Purpose |
|---|---|---|---|
| PREFIX Users | `{prefix} Users` | PREFIX Administrators + creator | All end users of the application |
| PREFIX Administrators | `{prefix} Administrators` | Creator | Developers and admins who manage design objects |

Properties for both auto-generated groups:
- Group Type: Custom
- Visibility: Restricted
- Membership: Closed
- Privacy Policy: Low
- Parent: None

### Extended Hierarchy for Role-Based Applications

For applications with multiple business roles, add role-specific groups:

```
PREFIX Users (auto-generated)
├── PREFIX Administrators (auto-generated, member of Users)
├── PREFIX Managers (custom, member of Users)
├── PREFIX Case Workers (custom, member of Users)
└── PREFIX Reviewers (custom, member of Users)
```

Create role groups with `parentGroupName` set to the PREFIX Users group name so they inherit base access. Store each group's reference in a constant for use in security expressions.

### Group Constants Pattern

Create constants for every group referenced in security expressions:

| Constant Name | Type | Value | Purpose |
|---|---|---|---|
| `PREFIX_ADMIN_GROUP` | GROUP | PREFIX Administrators group | Admin role checks |
| `PREFIX_ALL_USERS_GROUP` | GROUP | PREFIX Users group | Base user role checks |
| `PREFIX_MANAGER_GROUP` | GROUP | PREFIX Managers group | Manager role checks |
| `PREFIX_CASE_WORKER_GROUP` | GROUP | PREFIX Case Workers group | Case worker role checks |

## Per-Object-Type Security Configuration

### Applications

| Permission Level | Can Do |
|---|---|
| Administrator | Full control: update security, delete app, import/export, manage contents |
| Editor | Update properties/contents, import, delete app, view security |
| Viewer | View properties/contents, export, create/manage own packages |
| Deny | No access |

Application security controls who can manage the application itself. It does not control access to objects inside the application — each object has its own role map.

### Process Models

Security is configured at two levels:

1. **Role map** (inherited from application default security via `appUuid`):

| Permission Level | Can Do |
|---|---|
| Administrator | Full control over the process model definition |
| Manager | Manage running process instances |
| Editor | Edit the process model |
| Initiator | Start new process instances |
| Viewer | View the process model and running instances |
| Deny | No access |

2. **Security group** (`securityGroupName` parameter at creation):
   - The named group receives Initiator permissions
   - Must reference an existing group name
   - Choose based on who should start the process

Common patterns:
```
/* General user process — any app user can start */
securityGroupName: "CM Users"

/* Role-restricted process — only managers can start */
securityGroupName: "CM Managers"

/* Admin-only process — only admins can start */
securityGroupName: "CM Administrators"
```

### Sites

Site-level security comes from the application role map. Page-level security uses `visibilityExpr`:

| Configuration | How It Works |
|---|---|
| No `visibilityExpr` | Page visible to all users who can access the site |
| `visibilityExpr` set | Page visible only when expression evaluates to `true` for the user |

Common visibility expression patterns:

```
/* Admin-only page */
"if(a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_ADMIN_GROUP), true, false)"

/* Multiple roles */
"if(or(a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_ADMIN_GROUP), a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_MANAGER_GROUP)), true, false)"

/* All authenticated users (effectively no restriction) */
/* Omit visibilityExpr entirely */
```

### Folders

Auto-generated folders receive this security:

| Role | Permission |
|---|---|
| Default (All Other Users) | No Access |
| PREFIX Administrators | Administrator |
| PREFIX Users | Viewer |

Auto-generated folders:
- PREFIX Process Models (process model folder)
- PREFIX Rules & Constants (rule folder)
- PREFIX Knowledge Center (document knowledge center)
- PREFIX Artifacts (document folder, child of Knowledge Center)
- PREFIX Application Documentation (document folder, child of Knowledge Center)

Additional folders created with `appUuid` inherit the application's default security role map.

### Documents

Documents inherit security from their parent folder. No document-specific security configuration is needed in most cases. Upload documents to the appropriate folder to apply the correct access level.

### Record Types

Two security layers:

1. **Object-level** (role map via `appUuid`):

| Permission Level | Can Do |
|---|---|
| Administrator | Full control over record type definition |
| Editor | Edit record type configuration |
| Viewer | View records and use in queries |
| Deny | No access |

2. **Record-level** (row-level filtering):
   - Configured separately on the record type
   - Controls which individual records each user can see
   - Applied automatically to all queries against the record type

### Expression Rules, Interfaces, Constants

These objects use only the application default security role map:

| Permission Level | Can Do |
|---|---|
| Administrator | Full control (edit, delete, manage security) |
| Editor | Edit the object |
| Viewer | View and use the object |
| Deny | No access |

No object-specific security parameters exist for these types.

### Groups

Groups themselves have security:

| Permission Level | Can Do |
|---|---|
| Administrator | Manage group membership and properties |
| Editor | Edit group properties |
| Viewer | View group and its members |
| Deny | No access |

Groups are identified by name (not UUID) in the CLI. The `createGroup` operation accepts `parentGroupName` to establish hierarchy.

## Common Security Expression Patterns

### Role Check

```
a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_ROLE_GROUP)
```

### Multi-Role Check (OR)

```
or(
  a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_ADMIN_GROUP),
  a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_MANAGER_GROUP)
)
```

### Role + Data Condition (AND)

```
and(
  a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_CASE_WORKER_GROUP),
  rv!record[recordType!Case.fields.status] = cons!STATUS_OPEN
)
```

### Record Action Security (role-gated actions)

```
/* Only case managers can edit open cases */
and(
  a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_CASE_MANAGER_GROUP),
  or(
    rv!record[recordType!Case.fields.status] = cons!STATUS_OPEN,
    rv!record[recordType!Case.fields.status] = cons!STATUS_IN_PROGRESS
  )
)
```

### Record-Level Security Expressions

```
/* Owner-based: users see only their records */
a!queryFilter(
  field: recordType!Case.fields.assignedTo,
  operator: "=",
  value: loggedInUser()
)

/* Group-based: admins see all, others see only their own */
a!queryLogicalExpression(
  operator: "OR",
  filters: {
    a!queryFilter(
      field: recordType!Case.fields.assignedTo,
      operator: "=",
      value: loggedInUser()
    ),
    a!queryFilter(
      field: recordType!Case.relationships.employee.fields.manager,
      operator: "=",
      value: loggedInUser()
    )
  }
)

/* Status-based: only show active records */
a!queryFilter(
  field: recordType!Case.fields.status,
  operator: "<>",
  value: "Archived"
)
```

### Interface Conditional Visibility

```
/* Show section only to admins */
a!sectionLayout(
  label: "Admin Controls",
  showWhen: a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_ADMIN_GROUP),
  contents: { ... }
)

/* Show edit button only for open records */
a!buttonWidget(
  label: "Edit",
  showWhen: and(
    a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_CASE_MANAGER_GROUP),
    local!record[recordType!Case.fields.status] = cons!STATUS_OPEN
  )
)
```

---

## Group Management Confirmation Patterns

This section covers group-specific patterns for when to ask users for confirmation versus when to automatically complete mandatory steps. Always load `references/confirmation-patterns.md` for universal workflows.

**CRITICAL DIFFERENCE:** Groups are identified by **NAME** (not UUID) in most operations. This affects delete, rename, and member management operations.

### Group Deletion

**When to use:** Before deleting a group.

**Risk level:** CRITICAL if group has dependencies, HIGH otherwise

**Cross-reference:** `confirmation-patterns.md` → Universal Workflow 1 (Delete Confirmation)

#### Group Identification: NAME vs UUID

**IMPORTANT:** `deleteGroup` uses `groupName` (string), not UUID. This is different from other Appian objects.

**Why this matters:**
- Must verify group exists before delete (avoid deleting wrong group)
- Group names can be ambiguous ("Administrators" exists in multiple apps)
- Always confirm with user which specific group to delete

#### Dependency Checks

Before deleting a group, check for these dependencies:

1. **Constants referencing the group** (cannot detect automatically)
   - Search for constants of type GROUP
   - Common pattern: `PREFIX_ADMIN_GROUP`, `PREFIX_MANAGERS_GROUP`
   - Warn user: "If this group is referenced in constants, those constants will break. Manual verification needed."

2. **Child groups** via `listGroups()` filter by `parentGroupName`
   - If this group is a parent, deleting it orphans child groups
   - Child groups lose hierarchy structure
   - Child groups lose inherited permissions

3. **Security expressions** (cannot detect automatically)
   - Warn user: "This group may be used in security expressions on record types, interfaces, or process models. Manual verification needed via Appian Designer."

4. **Direct members** via `listGroupMembers(groupName)`
   - Count how many users are direct members
   - Members will lose permissions granted by this group

**Present to user based on findings:**

**If child groups exist (CRITICAL risk):**
```
You are about to DELETE group "[GroupName]".

This group has dependencies:
- [N] child groups (will become orphaned)
  - [ChildGroup1]
  - [ChildGroup2]
  - [...]
- [N] direct members
- May be referenced in GROUP constants (will become invalid references)
- May be used in security expressions (manual verification needed)

Impact:
- Child groups lose hierarchy structure and inherited permissions
- Direct members lose permissions granted by this group
- GROUP constants referencing this group will contain invalid group names
- Security expressions using this group will break

Recommendation: Delete or reassign child groups first, or delete entire hierarchy top-down.

This action CANNOT be undone.

To confirm, type: DELETE [GroupName]
```

**If no children (HIGH risk):**
```
You are about to DELETE group "[GroupName]".

This group has:
- [N] direct members
- May be referenced in GROUP constants (will become invalid references)
- May be used in security expressions (manual verification needed)

Continue? (yes/no)
```

### Group Rename

**When to use:** Before renaming a group via `updateGroup`.

**Risk level:** LOW (Appian automatically handles most references)

**IMPORTANT:** Despite the API using group names, groups are internally UUID-based. Renaming a group is SAFE for most operations.

#### What Appian Automatically Updates

When you rename a group, Appian automatically updates:
- ✅ **GROUP constants** - Value automatically changes to new name
- ✅ **Group memberships** - All member users and groups preserved (UUID-based internally)
- ✅ **Parent-child hierarchy** - Relationships preserved (UUID-based internally)
- ✅ **Group UUID** - Never changes (stable identifier)

#### What Requires Manual Updates

Only **hard-coded group names in SAIL expressions** need manual updates:
- ❌ Security expressions: `a!isUserMemberOfGroup(user, "OldName")` in interfaces
- ❌ SAIL code: References to `"OldName"` in expression rules
- ❌ Process models: Group name strings in conditions or assignments

#### Workflow

1. **Verify group exists** via `getGroup(groupName)`

2. **Search for hard-coded references** (cannot detect automatically)
   - Warn: "Search interfaces, expression rules, and process models for hard-coded group name strings"

3. **Present MEDIUM RISK confirmation:**
```
You are about to RENAME group "[OldName]" to "[NewName]".

What Appian will automatically update:
✅ GROUP constants (value will change to "[NewName]")
✅ Group memberships (all members preserved)
✅ Parent-child relationships (hierarchy preserved)

What requires manual updates:
❌ Hard-coded "[OldName]" in SAIL security expressions
❌ Hard-coded "[OldName]" in expression rules
❌ Hard-coded "[OldName]" in process model conditions

Recommendation: After rename, search for "[OldName]" in interfaces, expression rules, and process models. Update any hard-coded references to "[NewName]".

Continue with rename? (yes/no)
```

**After successful rename:**
```
✅ Group renamed from "[OldName]" to "[NewName]"

Automatic updates:
- GROUP constants updated
- Memberships preserved
- Hierarchy preserved

Manual action required:
Search for hard-coded "[OldName]" in:
1. Interfaces (security expressions)
2. Expression rules (a!isUserMemberOfGroup calls)
3. Process models (group name conditions)

Update any hard-coded references to "[NewName]".
```

### Parent Group Deletion

**When to use:** Before deleting a group that is a parent in the hierarchy.

**Risk level:** CRITICAL (affects hierarchy structure)

**Cross-reference:** Group Deletion workflow above

#### Special Considerations

Parent group deletion has cascading effects:

1. **Child groups become orphaned**
   - Lose parent relationship
   - Lose inherited permissions from parent
   - Still exist but hierarchy breaks

2. **Hierarchy flattens**
   - All child groups move to top level
   - No longer nested under parent

3. **Permission inheritance breaks**
   - Child groups no longer inherit parent's permissions
   - Must manually grant permissions to each child group

**Suggested alternatives before deleting parent:**

1. **Option A: Delete hierarchy top-down**
   - Delete children first
   - Then delete parent
   - Clean removal of entire branch

2. **Option B: Reassign children**
   - Move children to different parent via `updateGroup`
   - Set `parentGroupName` to different parent
   - Then delete original parent

3. **Option C: Preserve children**
   - Delete only the parent
   - Children become top-level groups
   - Manually grant necessary permissions to children

**Present options to user:**
```
You are about to DELETE parent group "[ParentName]".

This group has [N] child groups:
- [ChildGroup1]
- [ChildGroup2]
- [...]

These child groups will be orphaned (lose parent relationship and inherited permissions).

What would you like to do?
1. Delete parent only (children become top-level groups)
2. Delete entire hierarchy (delete children first, then parent)
3. Reassign children to different parent (move first, then delete)
4. Cancel

Choose an option: (1/2/3/4)
```

### Member Removal

**When to use:** Before removing a member from a group.

**Risk level:** MEDIUM (affects single user's permissions)

**Cross-reference:** `confirmation-patterns.md` → Universal Workflow 1 (Delete Confirmation)

#### Asymmetric Operations

**IMPORTANT:** Group member operations are asymmetric:
- `addGroupMembers` (plural) - batch operation, adds multiple users
- `removeGroupMember` (singular) - single operation, removes one user at a time

**Limitation:** `removeGroupMember` only affects **direct members**. It does NOT remove users who are members via group hierarchy (indirect membership).

#### Workflow

1. **Verify user is a direct member** via `listGroupMembers(groupName)`
   - Check if username appears in results
   - If not found, user may be indirect member (via child group)

2. **Check for critical roles:**
   - Is this the last administrator?
   - Is this a required role for production operations?

3. **Present confirmation:**

**Standard removal:**
```
Remove user "[username]" from group "[GroupName]"?

This will remove direct membership only. If user is a member via a child group, indirect access remains.

Continue? (yes/no)
```

**Critical role warning:**
```
You are about to REMOVE user "[username]" from group "[GroupName]".

WARNING: This is the last member of the Administrators group.

Removing this user will leave the group empty, and no one will have administrator permissions.

Are you sure? (yes/no)
```

#### Batch Removal Pattern

To remove multiple members, call `removeGroupMember` sequentially (not in parallel):

```
Removing 3 users from "[GroupName]":
1. Removing user1... ✅
2. Removing user2... ✅
3. Removing user3... ✅

All members removed successfully.
```
