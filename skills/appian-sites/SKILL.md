---
name: "appian-sites"
description: "Create and modify Appian sites — the user-facing web applications that organize pages, navigation, and branding for end users. Covers site creation, page configuration (Interface, Action, Record List types), page visibility expressions for role-based access, web address identifiers, and page ordering. Use when creating sites, adding or updating site pages, configuring navigation, or controlling page visibility."
---

## Relevant Tools

Site tools cover the full lifecycle:

- **Create a site** — provide a name, display name, web address identifier, and at least one page; optionally associate with an application via `appUuid`
- **List and get sites** — retrieve sites by UUID or search with optional filtering; scope to an application with `appUuid`
- **Update a site** — modify name, display name, web address identifier, description, or pages on an existing site; requires the full page list (update replaces all pages)
- **Delete a site** — permanently remove a site by UUID

## Creating Sites

### Naming Conventions

A site has three name-related fields:

- `name` — internal object name, used in Appian Designer. Use the application prefix: `PREFIX Site` (e.g., `ACME Site`, `CM Site`)
- `displayName` — user-facing name shown in the navigation menu and browser tab. Use a human-readable label: `Case Management`, `Employee Onboarding`, `Customer Portal`
- `webAddressIdentifier` — the URL slug that appears in the site's URL (e.g., `case-management` produces `/sites/case-management`). Use lowercase kebab-case, keep it short and descriptive. This value does not auto-update if you change the display name later.

### Required Parameters

Every site requires:

- `name` — internal object name
- `displayName` — user-facing display name
- `webAddressIdentifier` — URL slug for the site
- `pages` — at least one page definition (a site cannot be created empty)

### Typical Site Structure

Most Appian applications have a single site with multiple pages. A common pattern:

1. **Dashboard page** — an interface showing summary metrics, charts, and key indicators (often the first/default page)
2. **Record list pages** — one or more pages showing lists of records the user works with
3. **Action pages** — pages that let users initiate processes (e.g., "New Case", "Submit Request")
4. **Admin page** — an interface with administrative controls, visible only to administrators via a visibility expression

## Page Configuration

Each page in the `pages` array needs:

- `name` — page title displayed in the navigation bar and browser tab
- `targetUuid` — UUID of the interface to display on this page

Optional page properties:

- `description` — describes the page's purpose
- `iconId` — icon identifier displayed next to the page title in the navigation bar (e.g., `home`, `folder-open`, `cog`, `list`, `plus`, `chart-bar`, `users`). Defaults to `file-o` if omitted.
- `visibilityExpr` — a SAIL expression that controls whether the page is visible to the current user; the page is shown only when the expression evaluates to `true`. Omit to make the page visible to all users who can access the site.

### Page Types via the Tool Surface

The current tool surface creates pages that point to interfaces (`targetUuid`). This covers the most common page type — Interface pages. For other Appian page types (Action, Record List, Process HQ), the underlying platform supports them, but the tool surface uses `targetUuid` to reference an interface. To create a dashboard, record list view, or action landing page, build the content as an interface and point the site page to that interface.

### Page Ordering

Pages appear in the navigation bar in the order they are listed in the `pages` array. Put the most important page first — it becomes the default landing page when users open the site. A typical ordering:

1. Dashboard or home page
2. Primary record list pages
3. Secondary or supporting pages
4. Admin page (often last, with restricted visibility)

## Visibility Expressions

Use `visibilityExpr` to control which users can see a page. The expression must evaluate to `true` or `false` for the current user. Site-level access comes from the application's security role map; page-level visibility adds finer control within the site.

### Common Patterns

**Admin-only page:**
```
"if(a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_ADMIN_GROUP), true, false)"
```

**Multiple roles (OR):**
```
"if(or(a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_ADMIN_GROUP), a!isUserMemberOfGroup(loggedInUser(), cons!PREFIX_MANAGER_GROUP)), true, false)"
```

**All authenticated users (no restriction):**
Omit `visibilityExpr` entirely — the page is visible to everyone who can access the site.

### Prerequisites for Visibility Expressions

Visibility expressions typically reference group constants (e.g., `cons!PREFIX_ADMIN_GROUP`). These constants and the groups they reference must exist before creating the site. The dependency order is: Groups → Constants → Interfaces → Site.

## Web Address Identifier Conventions

- Use lowercase kebab-case: `case-management`, `employee-onboarding`, `hr-portal`
- Keep it short and meaningful — it appears in the URL and log files
- The identifier is set at creation and does not auto-update when you change the display name
- Must be unique across all sites in the environment
- Avoid special characters, spaces, or uppercase letters

## Updating Sites

### Read Before Update

Always retrieve the current site before updating. The update replaces the entire page list:

1. Get the site to see its current pages, display name, and configuration
2. Merge your changes with the existing state
3. Send the complete page list (existing pages + any additions or modifications)

### Adding a Page

To add a page to an existing site:

1. Get the current site to see its existing pages
2. Append the new page to the pages array
3. Send the full update with all pages (existing + new)

### Removing a Page

To remove a page, get the current site, remove the page from the array, and send the remaining pages. A site must always have at least one page.

### Reordering Pages

To change page order, get the current site, rearrange the pages array, and send the update. The first page in the array becomes the default landing page.

## Common Pitfalls

- **Creating a site with no pages** — a site requires at least one page; the create call will fail without it
- **Forgetting to include all existing pages on update** — the update replaces the entire page list; omitting existing pages removes them from the site
- **Interface doesn't exist yet** — the interface referenced by `targetUuid` must be created before the site references it; sites come late in the dependency order (after interfaces)
- **Web address identifier conflicts** — the identifier must be unique across the environment; if another site already uses it, creation fails
- **Visibility expression references missing constants** — if `visibilityExpr` references a constant that doesn't exist, the page may error or behave unexpectedly; create groups and constants before the site
- **Changing display name and expecting URL to update** — the web address identifier is independent of the display name; update it explicitly if the URL needs to change
- **Using long-running expressions in display name** — if the display name is an expression, it evaluates whenever sites load; avoid queries or expensive computations
- **Not associating the site with the application** — pass `appUuid` when creating the site so it inherits the application's default security and appears in the application's object list

## When You Need More

For SAIL syntax used in the interfaces that power site pages, activate the SAIL knowledge skill. For security patterns including group hierarchy design and visibility expression patterns, activate the security knowledge skill.

For questions about advanced site configuration, branding, navigation bar styles, or platform capabilities beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.
