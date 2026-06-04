---
name: "appian-supporting-objects"
description: "Create and manage Appian constants, groups, folders, and documents — the utility objects that support all other Appian design objects. Covers constant types and naming, group hierarchy and parent relationships, folder types and organization, and document upload and content management. Use when creating constants, managing groups, organizing folders, or working with documents."
---

## Constants

### Relevant Tools

Constant tools cover the full lifecycle:

- **Create a constant** — provide a name, type, and value; optionally include a description; associate with an application via `appUuid`
- **List and get constants** — retrieve constants by UUID or search with optional filtering; scope to an application with `appUuid`
- **Update a constant** — modify name, description, type, or value on an existing constant
- **Delete a constant** — permanently remove a constant by UUID

### Naming Conventions

- **Pattern**: `PREFIX_DESCRIPTIVE_NAME` in UPPER_SNAKE_CASE
- **Prefix**: Use the application prefix (e.g., `CM_`, `ACME_`, `HR_`)
- **Name should describe what the constant holds**, not how it's used
- **Examples**:
  - `CM_ADMIN_GROUP` — stores a reference to the administrators group
  - `CM_STATUS_OPEN` — stores the text value "Open" for case status
  - `CM_MAX_ATTACHMENTS` — stores the maximum number of allowed attachments
  - `CM_DEFAULT_SLA_DAYS` — stores the default SLA duration in days
  - `CM_SUPPORT_EMAIL` — stores a support email address

### Constant Types

| Type | Description | Typical Use |
|---|---|---|
| `TEXT` | String value | Status labels, configuration strings, email addresses |
| `NUMBER` | Numeric value (decimal) | Thresholds, percentages, rates |
| `INTEGER` | Whole number | Counts, limits, SLA days |
| `BOOLEAN` | True/false | Feature flags, toggles |
| `DATE` | Date value | Cutoff dates, policy effective dates |
| `DATETIME` | Date and time value | Scheduled timestamps |
| `TIME` | Time value | Business hours, deadlines |
| `USER` | Appian user reference | System accounts, default assignees |
| `GROUP` | Appian group reference | Security group references for visibility expressions |
| `FOLDER` | Appian folder reference | Target folders for document uploads |
| `DOCUMENT` | Appian document reference | Template documents, reference files |

### Common Use Cases

**Group references** — Store group IDs in constants so security expressions stay readable and maintainable. Every group created for role-based access should have a corresponding constant:
```
cons!CM_ADMIN_GROUP
cons!CM_CASE_MANAGER_GROUP
cons!CM_ALL_USERS_GROUP
```

**Status values** — Store status labels as TEXT constants to avoid hardcoded strings scattered across interfaces and process models:
```
cons!CM_STATUS_OPEN
cons!CM_STATUS_IN_PROGRESS
cons!CM_STATUS_CLOSED
```

**Configuration values** — Store thresholds, limits, and settings that may change over time:
```
cons!CM_MAX_ATTACHMENTS     /* INTEGER: 10 */
cons!CM_DEFAULT_SLA_DAYS    /* INTEGER: 5 */
cons!CM_SUPPORT_EMAIL       /* TEXT: "support@example.com" */
```

### Pitfalls

- **Not using the application prefix** — constants without a prefix are hard to find and may collide with constants from other applications in the same environment
- **Using lowercase or mixed case** — constant names should always be UPPER_SNAKE_CASE for consistency and to distinguish them from other object types
- **Hardcoding values instead of using constants** — if a value appears in multiple places (group IDs, status labels, configuration), extract it to a constant; this makes updates a single-point change
- **Wrong type for the value** — the type must match the value; storing a group reference as TEXT instead of GROUP means it won't work in group-based functions like `a!isUserMemberOfGroup()`
- **Not associating with the application** — pass `appUuid` when creating constants so they inherit the application's default security and appear in the application's object list

---

## Groups

### Relevant Tools

Group tools cover creation and discovery:

- **Create a group** — provide a name and optionally a parent group name and description; associate with an application via `appUuid`
- **List groups** — retrieve groups with optional filtering; scope to an application with `appUuid`
- **Get a group** — retrieve a single group by name

### Application Default Groups

When you create an application, Appian auto-generates two groups:

- **PREFIX Administrators** — developers and admins who manage the application
- **PREFIX Users** — all end users (includes Administrators as members)

These groups form the application's default security role map. All objects created with the application's `appUuid` inherit this role map automatically.

### Creating Additional Groups

Create groups beyond the auto-generated pair when a concrete security requirement demands it — different record-level access, role-restricted site pages, or role-gated process actions.

Each additional group needs:

- `name` — follow the `PREFIX RoleName` convention in Title Case (e.g., `CM Case Managers`, `CM Supervisors`, `CM Approvers`)
- `parentGroupName` — (optional) name of an existing group to nest under; typically set to the PREFIX Users group so role groups inherit base-level access
- `description` — (optional) describes the group's purpose and who belongs to it

### Hierarchy Patterns

- Make role-specific groups children of PREFIX Users so they inherit base access
- Keep the hierarchy shallow — one level of role groups under PREFIX Users is usually sufficient
- Store group references in constants immediately after creating groups (e.g., `cons!CM_CASE_MANAGER_GROUP` with type GROUP)

### Naming Conventions

- **Pattern**: `PREFIX RoleName` in Title Case
- **Examples**: `CM Case Managers`, `CM Supervisors`, `CM All Users`, `HR Recruiters`, `ACME Approvers`
- The prefix matches the application prefix used across all objects

### Pitfalls

- **Creating groups speculatively** — start with the auto-generated Administrators and Users groups; add role groups only when a concrete security requirement demands it
- **Forgetting to set parentGroupName** — without a parent, the group is a top-level group disconnected from the application's hierarchy; role groups should typically be children of PREFIX Users
- **Mismatched group name in references** — the `securityGroupName` on process models and group references in constants must match the group name exactly (case-sensitive)
- **Not creating corresponding constants** — after creating a group, create a GROUP-type constant that references it; this keeps security expressions readable (`cons!CM_ADMIN_GROUP` instead of hardcoded group IDs)
- **Not associating with the application** — pass `appUuid` when creating groups so they appear in the application's object list

---

## Folders

### Relevant Tools

Folder tools cover the full lifecycle:

- **Create a folder** — provide a name and parent folder UUID; optionally include a description; associate with an application via `appUuid`
- **List and get folders** — retrieve folders by UUID or search with optional filtering; scope to an application with `appUuid`; filter by folder type
- **List folder contents** — retrieve the contents of a folder by UUID
- **Update a folder** — modify name or description on an existing folder
- **Delete a folder** — permanently remove a folder by UUID

### Folder Types

Appian has distinct folder types that serve different purposes:

- **Rule folders** — contain expression rules, interfaces, constants, and other rule-type objects. Created automatically when an application is created (e.g., "PREFIX Rules & Constants").
- **Document folders (Knowledge Centers)** — contain documents and sub-folders for document storage. Created automatically when an application is created (e.g., "PREFIX Knowledge Center").
- **Process model folders** — contain process models. Process models can only live in process model folders, not in regular folders. Created automatically when an application is created.

### Application Default Folders

When you create an application, Appian auto-generates default folders:

- A rules folder for expression rules, interfaces, and constants
- A document folder (Knowledge Center) for documents
- A process model folder for process models

Discover these by listing folders scoped to the application with `appUuid`.

### Creating Additional Folders

Create additional folders when you need to organize objects within the default structure:

- **Sub-folders under the Knowledge Center** — organize documents by category (e.g., "Templates", "Reports", "Attachments")
- **Sub-folders under the rules folder** — organize rules and interfaces by feature area in large applications

Every folder requires a `parentFolderUuid` — you cannot create a top-level folder without a parent. Discover existing folders first to find the appropriate parent.

### Naming Conventions

- **Pattern**: `PREFIX FolderName` in Title Case for top-level folders; descriptive names for sub-folders
- **Examples**: `CM Templates`, `CM Reports`, `Case Attachments`, `Email Templates`

### Pitfalls

- **Putting process models in regular folders** — process models require process model folders; use the process model folder listing tool to find valid parent folders for process models
- **Creating folders without a parent** — every folder needs a `parentFolderUuid`; discover existing folders first
- **Not discovering existing folders before creating** — the application may already have default folders; list folders scoped to the application before creating new ones to avoid duplicates
- **Not associating with the application** — pass `appUuid` when creating folders so they inherit the application's default security

---

## Documents

### Relevant Tools

Document tools cover the full lifecycle:

- **Upload a document** — provide a name, parent folder UUID, and content; optionally include a description and file extension; associate with an application via `appUuid`
- **List and get documents** — retrieve documents by UUID or search with optional filtering; scope to an application with `appUuid`
- **Update a document** — modify name or description on an existing document
- **Delete a document** — permanently remove a document by UUID
- **Get document content** — retrieve the binary content of a document
- **Get document text** — extract the text content of a document, optionally including metadata
- **Replace document content** — replace the content of an existing document with new content

### Uploading Documents

Every document upload requires:

- `name` — the document name including file extension (e.g., `Annual Report.pdf`, `case-template.xlsx`). Maximum 255 characters.
- `parentFolderUuid` — UUID of the document folder (Knowledge Center or sub-folder) to upload into
- `content` — the file content as a plain string (automatically base64-encoded by the tool)

Optional parameters:

- `description` — describes the document's purpose
- `extension` — file extension without the dot (e.g., `pdf`, `xlsx`); auto-detected from the name if not provided
- `appUuid` — application UUID; associates the document with the application and applies default security

### Content Management

- **Replace content** — update an existing document's content without changing its metadata; useful for template updates or version refreshes
- **Extract text** — retrieve the text content of a document for processing or analysis; optionally include document metadata
- **Get binary content** — retrieve the raw binary content of a document

### Naming Conventions

- Include the file extension in the name: `case-template.xlsx`, `welcome-email.html`, `logo.png`
- Use descriptive names that indicate the document's purpose
- For templates, prefix with the application prefix: `CM Case Report Template.docx`

### Document Security

Documents inherit security from their parent folder. Upload documents to the appropriate folder and the folder's security applies automatically. No additional document-level security configuration is typically needed.

### Pitfalls

- **Uploading to the wrong folder type** — documents must go in document folders (Knowledge Centers), not rule folders or process model folders
- **Exceeding the 255-character name limit** — document names including extension must be 255 characters or fewer
- **Forgetting the file extension in the name** — include the extension in the `name` field (e.g., `report.pdf`); if omitted and `extension` is not provided, the document may not have a recognizable type
- **Not discovering existing folders before uploading** — find the appropriate document folder (Knowledge Center) by listing folders scoped to the application
- **Not associating with the application** — pass `appUuid` when uploading documents so they inherit the application's default security

---

## Dependency Position

Supporting objects appear early in the Appian object dependency order:

1. **Application** (creates default groups and folders)
2. **Groups** (additional role groups, if needed)
3. **Folders** (additional sub-folders, if needed)
4. **Constants** (reference groups, store configuration values)

Groups, folders, and constants must exist before the objects that reference them — record types, expression rules, interfaces, process models, sites, and web APIs all may reference constants and depend on the folder and group structure being in place.

Documents come last in the dependency order (step 12) because they are uploaded to folders and rarely referenced by other design objects during creation.

## When You Need More

For security patterns including group hierarchy design, role-based access, and visibility expressions that reference group constants, activate the security knowledge skill. For expression syntax used in constant references (`cons!`), activate the expressions knowledge skill.

For questions about advanced folder security, document versioning, or platform capabilities beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.
