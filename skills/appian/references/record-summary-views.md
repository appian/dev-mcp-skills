# Record Summary Views

## Purpose
Record summary views display detailed information about a single record instance. They are accessed via links from record lists or custom grids, not as standalone site pages.

## Scope
Define summary views **only for primary entity record types** that require detail views:
- ✅ **Include:** Case, Application, Request, Customer, Project - entities users navigate to and view details
- ❌ **Exclude:** Status, Priority, Category, Type - reference/lookup record types that don't need dedicated views

## Summary View Definition
For each record type that needs a detail view, capture:
- **View Name:** Following the pattern `[Entity] Summary View`
- **Record Type:** The record type this view displays
- **Interface Name:** The interface used for this view (dedicated summary interface)
- **Sections Displayed:** MUST match the sections defined in the corresponding summary view interface. Use `["Content"]` when requirements don't specify section structure, or list specific section names when requirements explicitly describe distinct areas. This field mirrors the interface's actual sections to maintain consistency.
- **Related Records Displayed:** Which related record types are shown (e.g., "Case Notes," "Attachments")
- **Requirements Source:** Citation from requirements document indicating users need to view details of this entity

## Summary View Design Guidelines

1. **Naming Convention:** Use `[Entity] Summary View` pattern
   - Example: `Application Summary View`, `Case Summary View`

2. **Separate from Forms:** Create dedicated summary view interfaces rather than reusing create/update forms in read-only mode. Summary views often show different information (e.g., related records, audit history) not present in forms.

3. **Not Site Pages:** Summary views are accessed via record links, not as direct site pages. They appear when users click a record from a grid or list.

4. **Include Related Records:** Summary views typically display the parent record plus grids of related child records (e.g., Case details + Case Notes grid + Attachments grid).

5. **Primary Entities Only:** Only create summary views for record types that users navigate to and view. Reference record types (Status, Priority, Category, etc.) do not need summary views.

6. **Summary Views are Interfaces:** Summary views are defined as interfaces in the `interfaces` array with `interface_type: "Record View"`.

## Related Record Views

Related record views display data from separate record types that are related to the primary record.

**When to Create Related Record Views:**
- Requirements explicitly mention viewing related record type data
- Related data is a separate record type (not embedded attributes)
- Related data is NOT core to the primary entity (e.g., Case History is separate, but Case Notes are core)

**Examples:**
- ✅ Customer record → Customer Contracts view (separate CONTRACT record type)
- ✅ Case record → Case History view (separate AUDIT_LOG record type)
- ❌ Case record → Case Notes (Notes are core attributes of CASE, not separate record type)

**Note:** Most applications only need Summary View. Create related views only when requirements explicitly describe viewing related record type data.

## Interface Types Reference

| Type | Description | Primary Usage | Examples |
|------|-------------|---------------|----------|
| **Landing Page** | Main entry point after login. Shows overview, recent activity, quick actions, navigation. | Site page (Interface type) | Applicant Dashboard, Admin Home |
| **Form** | Data entry for creating/updating records. Use `isUpdate` parameter to handle both modes in one interface. | Start form (process models) or task forms | Application Form, User Registration Form |
| **Wizard** | Multi-step form interface using `a!wizardLayout()`. Interface type signals UX component choice but does NOT define step breakdown - that's a separate design activity. List all fields in single "Content" section. | Start form (process models) | Order Entry Wizard, Onboarding Wizard |
| **Dashboard** | Metrics, charts, analytics in tiles/cards. May include embedded record grids. | Site page (Interface type) | Review Metrics, Statistics Dashboard |
| **Report** | Formatted for printing/exporting to PDF/Excel. | Site page (Report type) or embedded | Monthly Statistics, Audit Trail |
| **Record View** | Single-record view with related data. Supports multiple views per entity (Summary, Audit Trail, etc.). Accessed via record links. | Record type view config | Application Details, Case Details, Audit Trail |

### ~~Record List~~ (DEPRECATED)
Do not use as standalone type. Instead: embed custom grids in Landing Page/Dashboard interfaces, or use native Record List site pages (rare).
