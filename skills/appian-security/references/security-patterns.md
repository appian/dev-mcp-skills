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

Groups are identified by name (not UUID) in the API. The `createGroup` operation accepts `parentGroupName` to establish hierarchy.

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
