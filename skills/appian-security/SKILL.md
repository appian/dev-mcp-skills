---
name: "appian-security"
description: "Design and configure Appian security — group hierarchies, role-based access, object permissions, and record-level security. Use when setting up security, creating groups, configuring access controls, writing visibility expressions, or designing role-based access patterns for any Appian application."
---

## Group Hierarchy Design

Every Appian application needs a group hierarchy that maps business roles to platform permissions. Design groups before creating other objects — groups are referenced by process models, sites, folders, and security expressions throughout the application.

### Standard Hierarchy

When you create an application, Appian auto-generates two groups using the application prefix:

- **PREFIX Administrators** — developers and admins who manage the application's design objects
- **PREFIX Users** — all end users of the application (includes Administrators as members)

These become the application's default security groups. All objects created with the application's `appUuid` inherit this role map automatically.

For applications with role-based access beyond admin/user, create additional groups as children of the application-level groups:

1. Start with the auto-generated groups (PREFIX Administrators, PREFIX Users)
2. Create role-specific groups for each business role (e.g., PREFIX Case Managers, PREFIX Reviewers, PREFIX Approvers)
3. Make role groups members of PREFIX Users so they inherit base-level access
4. Store group references in constants (e.g., `cons!CASE_MANAGER_GROUP`) so security expressions stay readable and maintainable

### Group Naming

Follow the application prefix convention: `PREFIX RoleName`. Examples:
- `CM Case Managers` (for a Case Management app with prefix CM)
- `CM Supervisors`
- `CM All Users`

### When to Create Additional Groups

Create groups beyond the auto-generated pair when:
- Different users need different record-level access (e.g., managers see all cases, agents see only assigned cases)
- Site pages should be visible only to certain roles
- Process model actions are restricted to specific roles
- Record actions (create, update, delete) are role-gated

Do not create groups speculatively. Start with the auto-generated Administrators and Users groups; add role groups only when a concrete security requirement demands it.

## Default Security Propagation

When you pass `appUuid` to any object creation operation, Appian automatically:

1. Adds the new object to the application
2. Applies the application's default security role map to the object

This means objects created with `appUuid` inherit the PREFIX Administrators (Administrator permission) and PREFIX Users (Viewer permission) role map without any additional configuration.

Always pass `appUuid` when creating objects that belong to an application. This ensures consistent baseline security across all objects and avoids orphaned objects with no security.

### What Propagation Does NOT Do

- It does not set object-specific security parameters (like `securityGroupName` on process models or `visibilityExpr` on site pages)
- It does not configure record-level security on record types
- It does not override security you explicitly set — it provides the baseline

You still need to configure object-specific security where the object type supports it.

## Object-Level Security Patterns

Different object types have different security mechanisms. Here's how to configure security for each type.

### Process Models

Process models require a `securityGroupName` parameter at creation time. This group gets Initiator permissions — meaning members of that group can start the process.

- Use the PREFIX Users group (or a role-specific group) as the security group
- The security group name must match an existing group name exactly
- The application's default security role map is applied separately via `appUuid`

```
/* When creating a process model, specify who can initiate it */
securityGroupName: "CM Case Managers"
```

Choose the security group based on who should be able to start the process:
- General processes (e.g., submit a case): use PREFIX Users
- Role-restricted processes (e.g., approve a case): use the specific role group
- Admin-only processes (e.g., bulk data operations): use PREFIX Administrators

### Sites and Page Visibility

Sites control access at the page level using `visibilityExpr` — a SAIL expression that evaluates to Boolean for each user. Pages without a visibility expression are visible to all users who can access the site.

```
/* Restrict a page to administrators only */
visibilityExpr: "if(a!isUserMemberOfGroup(loggedInUser(), cons!CM_ADMIN_GROUP), true, false)"

/* Restrict a page to a specific role */
visibilityExpr: "if(a!isUserMemberOfGroup(loggedInUser(), cons!CM_CASE_MANAGER_GROUP), true, false)"
```

Use visibility expressions when:
- A site has pages for different roles (e.g., admin dashboard vs. user dashboard)
- Certain pages should only appear for managers or supervisors
- You need to hide configuration or reporting pages from end users

Store group references in constants rather than hardcoding group IDs in visibility expressions.

### Folders

Folders inherit security from the application's default role map when created with `appUuid`. The auto-generated folders (Process Models, Rules & Constants, Knowledge Center, etc.) are pre-configured with:
- Default (All Other Users): No Access
- PREFIX Administrators: Administrator
- PREFIX Users: Viewer

For additional folders, creating them with `appUuid` applies the same baseline. Adjust folder security only when you need to restrict developer access to specific folders.

### Documents

Documents inherit security from their parent folder. Upload documents to the appropriate folder and the folder's security applies automatically. No additional document-level security configuration is typically needed.

### Expression Rules, Interfaces, Constants

These objects rely entirely on the application's default security role map (applied via `appUuid`). They don't have object-specific security parameters. The role map determines who can view, edit, and administer each object.

### Record Types

Record types have two layers of security:
1. **Object-level security** — who can see the record type definition (controlled by the role map, applied via `appUuid`)
2. **Record-level security** — who can see which individual records (configured separately, see below)

## Record-Level Security

By default, any user with Viewer permission to a record type can see all records. Record-level security adds row-level filtering so different users see different subsets of data.

### When to Configure Record-Level Security

Configure it when:
- Different roles should see different records (e.g., agents see only their assigned cases)
- Sensitive data should be restricted to specific groups
- Users should only see records related to their department, region, or team

Skip it when all users with access to the record type should see all records.

### Security Expressions

Use `a!queryFilter()` or `a!queryLogicalExpression()` to define filter conditions. These filters are automatically applied everywhere the record type is queried — grids, reports, record lists, related records.

```
/* Users can only see records assigned to them */
a!queryFilter(
  field: recordType!Case.fields.assignedTo,
  operator: "=",
  value: loggedInUser()
)

/* Users can see records assigned to them OR records they manage */
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
```

### Security Rules vs. Security Expressions

Appian offers two approaches to record-level security:
- **Security rules** — a guided, no-code configuration experience. Preferred for most use cases. Supports inheritance from related record types.
- **Security expressions** — expression-based filter conditions for complex logic. Cannot be combined with security rules on the same record type.

Use security rules when possible. Use security expressions when the logic requires complex joins or conditions that the guided experience doesn't support.

### Common Record-Level Security Patterns

- **Owner-based**: users see only records where they are the owner/assignee (`field = loggedInUser()`)
- **Group-based**: members of a group see all records, others see only their own (combine group check with owner filter using OR)
- **Status-based**: restrict visibility based on record status (e.g., draft records visible only to creator)
- **Hierarchy-based**: managers see their direct reports' records (use relationship fields)

## Role-Based Access Patterns

Map business roles to Appian groups, then use those groups consistently across all security touchpoints.

### Security Expression Functions

Use these functions in visibility expressions, record action security, and interface conditional logic:

```
/* Check if the current user is in a group */
a!isUserMemberOfGroup(loggedInUser(), cons!CASE_MANAGER_GROUP)

/* Check membership in multiple groups */
or(
  a!isUserMemberOfGroup(loggedInUser(), cons!ADMIN_GROUP),
  a!isUserMemberOfGroup(loggedInUser(), cons!SUPERVISOR_GROUP)
)

/* Combined role + data check */
and(
  a!isUserMemberOfGroup(loggedInUser(), cons!CASE_MANAGER_GROUP),
  rv!record[recordType!Case.fields.status] = cons!STATUS_OPEN
)
```

### Where Role Checks Apply

- **Site page visibility** — `visibilityExpr` on site pages
- **Interface conditional visibility** — `showWhen` on components and sections
- **Record action security** — expressions controlling who can execute record actions
- **Process model security** — `securityGroupName` for process initiation
- **XOR gateway conditions** — routing process flow based on the initiator's role

### Constants for Group References

Always store group references in constants rather than hardcoding group names or IDs:

```
cons!CM_ADMIN_GROUP        /* PREFIX Administrators group */
cons!CM_ALL_USERS_GROUP    /* PREFIX Users group */
cons!CM_CASE_MANAGER_GROUP /* Role-specific group */
cons!CM_SUPERVISOR_GROUP   /* Role-specific group */
```

This makes security expressions readable and allows group references to be updated in one place.

## Least-Privilege Principles

- Start with the auto-generated Administrators and Users groups — don't over-engineer the group hierarchy upfront
- Grant Viewer permissions by default; escalate to Editor or Administrator only when needed
- Use the Deny permission level sparingly — it overrides all other permissions for that group, which can cause confusing behavior
- Restrict process model initiation to the narrowest appropriate group
- Configure record-level security to show users only the data they need
- Use site page visibility to hide admin-only pages from regular users
- When in doubt, restrict access and widen it later — it's easier to grant access than to revoke it

## Pitfalls

- Forgetting to pass `appUuid` when creating objects — the object gets no default security and may be inaccessible or overly accessible
- Using user-specific security instead of group-based security — Appian recommends always using groups in role maps, never individual users
- Hardcoding group names in expressions instead of using constants — makes maintenance difficult and error-prone
- Setting `securityGroupName` on a process model to a group that doesn't exist — the creation will fail
- Confusing application security with object security — the application's role map controls who can manage the application itself, not the objects inside it. Each object has its own role map.
- Forgetting that users in multiple groups get the highest permission — if a user is in both a Viewer group and an Administrator group, they get Administrator access
- Not configuring record-level security when different roles need different data access — by default, all Viewers see all records
- Using security expressions when security rules would suffice — security rules are easier to maintain and support inheritance from related record types

## When You Need More

For questions about specific permission levels, advanced record-level security patterns, or security configuration beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.

Load `references/security-patterns.md` when you need group hierarchy templates and per-object-type security configuration details.
