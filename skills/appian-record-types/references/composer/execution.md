## Record Type Implementation Guide

**Scope:** This guide covers creating database-backed record types where Appian creates and manages the database table.

When creating a record type, always use:
- `sourceType: "DATABASE"`
- `createTable: true` (Appian creates and manages the table)

**Out of Scope:**
- WEB_SERVICE, PROCESS, SALESFORCE record types → Not supported
- DATABASE with createTable=false (connecting to existing external tables) → Not supported
- For information on these types, see: 
  - https://docs.appian.com/suite/help/26.4/Record_Type_Object.html
  - https://docs.appian.com/suite/help/26.4/configure-record-data-source.html

Record types are created one at a time. There is no batch operation.

### Creation Order

Create record types sequentially — reference/lookup tables first, then entity tables that depend on them via foreign keys.

Every record type must have a primary key field. The primary key field MUST be named `id` and be of type `INTEGER`.

**Important:** Appian does not support compound primary keys. Junction tables like CUSTOMER_USE_CASE cannot use (customerId, useCaseId) as the primary key. Always use a single `id` field as PK, even for junction tables.

### Record Type and Field Descriptions

When creating record types and fields, ALWAYS include descriptions when available from the planning phase's implementation_notes.

**Record Type Description**:
- Check implementation_notes for "Description: [text]"
- Pass the description text to the `description` parameter of createRecordType
- Strip any labeling markers ([PROPOSED], [DERIVED], (Source: Requirements)) before passing to tool
- This makes the record type self-documenting in Appian Designer

**Field Descriptions**:
- Check implementation_notes for field descriptions (format: "fieldName (TYPE, required/optional) - description text")
- If a field has a description (text after the hyphen), pass it to the `description` parameter of addRecordTypeField
- Strip any labeling markers ([PROPOSED], [DERIVED], (Source: Requirements)) before passing to tool
- Only include descriptions that were documented in planning phase; don't fabricate them
- If no description is present in implementation_notes (e.g., "name (TEXT, required)"), omit the description parameter

**Label Stripping Pattern**:
Remove these markers before passing to tools:
- `[PROPOSED]` or `[PROPOSED - ...]`
- `[DERIVED]` or `[DERIVED from ...]`
- `(Source: Requirements - "...")`

**Example implementation_notes** (from planning phase):
```
Description: Represents governance committees within the organization [PROPOSED - please verify]
Fields:
- id (INTEGER PK)
- name (TEXT, required)  ← No description = omit description parameter
- isActive (BOOLEAN, required, default: true) - Whether the committee appears in active dropdowns [PROPOSED]
- meetingFrequency (TEXT, optional) - How often the committee meets (Monthly, Quarterly) [PROPOSED]
- accountCode (TEXT, required) - Must be exactly 8 characters (Source: Requirements - "account codes are 8 chars")
```

Note: Text after the hyphen is the field description. [PROPOSED], [DERIVED], and (Source: Requirements) are planning markers to strip.

**Corresponding tool calls** (markers stripped):
```python
# Create record type with description (stripped [PROPOSED] marker)
createRecordType(
    name="COMMITTEE",
    description="Represents governance committees within the organization",
    ...
)

# Field without description = omit description parameter
addRecordTypeField(
    uuid=committee_uuid,
    fieldName="name",
    fieldType="TEXT"
)

# Field with description (stripped [PROPOSED] marker)
addRecordTypeField(
    uuid=committee_uuid,
    fieldName="isActive",
    fieldType="BOOLEAN",
    description="Whether the committee appears in active dropdowns"
)

# Field with description (stripped [PROPOSED] marker)
addRecordTypeField(
    uuid=committee_uuid,
    fieldName="meetingFrequency",
    fieldType="TEXT",
    description="How often the committee meets (Monthly, Quarterly)"
)

# Field with description from user requirements (stripped (Source: Requirements) marker)
addRecordTypeField(
    uuid=committee_uuid,
    fieldName="accountCode",
    fieldType="TEXT",
    description="Must be exactly 8 characters"
)

# Long Text field (descriptions, notes, comments) — use TEXT with length: 4000
addRecordTypeField(
    uuid=committee_uuid,
    fieldName="description",
    fieldType="TEXT",
    length=4000,
    description="Detailed description of the committee's purpose"
)

# Extra Long Text field (rich content over 4000 chars) — use EXTRA_LONG_TEXT
addRecordTypeField(
    uuid=committee_uuid,
    fieldName="meetingNotes",
    fieldType="EXTRA_LONG_TEXT",
    description="Full meeting notes and minutes"
)
```

Important: Always strip labeling markers ([PROPOSED], [DERIVED], (Source: Requirements)) before passing descriptions to tools. These markers are for planning review only and should not appear in Appian Designer.

These descriptions improve Appian Designer usability and provide context for understanding the data model.

### Field Ordering

Fields are displayed in the UI in the order they're added. Add fields in this sequence:

1. **Primary key:** id (INTEGER)
2. **Descriptive fields:** name, title, description
3. **Classification fields:** statusId, priorityId, categoryId (FKs to reference tables)
4. **Temporal fields:** requestDate, dueDate, completionDate, startDate
5. **Quantitative fields:** amount, quantity, cost, price
6. **User reference fields:** requestedBy, assignedTo, approvedBy, reviewedBy (USER type)
7. **Audit fields:** createdAt, createdBy, modifiedAt, modifiedBy
8. **Soft delete fields** (if applicable): isDeleted, deletedAt, deletedBy

This order matches the field list structure from the planning phase (data_model.md) and creates a logical flow in form interfaces.

### Unique Constraints

When implementation_notes documents single-field uniqueness, apply it during field creation using the `isUnique` parameter.

**Single-Field Uniqueness:**

If implementation_notes contains patterns like:
- "UNIQUE constraint: email"
- "UNIQUE constraint: employeeNumber"
- "UNIQUE constraint: licensePlate"

Then call `addRecordTypeField` with `isUnique=True`:

```python
addRecordTypeField(
    uuid=record_type_uuid,
    fieldName="email",
    fieldType="TEXT",
    isUnique=True  # Enforces uniqueness at database level
)
```

**Example from implementation_notes:**
```
CUSTOMER:
Fields: id (INTEGER PK), name (TEXT), email (TEXT), phone (TEXT)
UNIQUE constraint: email
```

**Execution:**
```python
# Create record type
customer_uuid = createRecordType(name="CUSTOMER", ...)

# Add fields
addRecordTypeField(uuid=customer_uuid, fieldName="id", fieldType="INTEGER", isPrimaryKey=True)
addRecordTypeField(uuid=customer_uuid, fieldName="name", fieldType="TEXT")
addRecordTypeField(uuid=customer_uuid, fieldName="email", fieldType="TEXT", isUnique=True)  # ← Applied
addRecordTypeField(uuid=customer_uuid, fieldName="phone", fieldType="TEXT")
```

**Compound Uniqueness NOT Supported:**

If implementation_notes documents compound uniqueness like:
- "UNIQUE constraint: (studentId, courseId)"
- "UNIQUE constraint: (orderId, lineNumber)"

The per-field `isUnique` parameter cannot express this (it only works for single fields). These constraints are documented in planning but **cannot be enforced at database level** during execution. Application logic must prevent duplicates.

**Important:** Only apply `isUnique=True` when implementation_notes explicitly documents it. Do not infer uniqueness from field names or descriptions.

### Relationships

Relationships cannot be defined during record type creation because field UUIDs are not assigned until the record type exists. Always create the record type first, then wire relationships afterward.

After creating each record type, check whether any relationships can now be wired — meaning both the source and target record types already exist (from this or earlier tasks). Wire all applicable relationships before marking the task complete. Do not list "add relationships" as separate plan steps — they are part of the record type creation task.

### Relationship Workflow

1. Create reference/lookup record types first — save the returned UUIDs and field UUIDs
2. Create entity record types with foreign key fields — save the returned UUIDs and field UUIDs
3. Wire relationships using the real field UUIDs from the creation responses

When wiring a relationship, you need three pieces of information:
- The foreign key field UUID on the source record type (from its creation response)
- The primary key field UUID on the target record type (from its creation response)
- The UUID of the target record type itself

**NEVER guess or fabricate field UUIDs.** Always use the exact UUIDs returned when the record type was created or retrieved.

**Check the "OBJECTS CREATED SO FAR" section first** — it contains field UUIDs for all record types created in earlier tasks. Use those directly. Only retrieve a record type's fields if it was created outside this plan and is not in the structured context.

### Junction Tables

Junction tables (for many-to-many relationships) require FOUR bidirectional relationships to be wired — two for each foreign key field:

Example: STUDENT_COURSE junction table
- Has fields: id (PK), studentId (FK), courseId (FK), enrolledDate
- Wire relationship 1: studentId → STUDENT.id (MANY_TO_ONE) - "Get student for this enrollment"
- Wire relationship 2: STUDENT.id → studentId (ONE_TO_MANY) - "Get all enrollments for this student"
- Wire relationship 3: courseId → COURSE.id (MANY_TO_ONE) - "Get course for this enrollment"
- Wire relationship 4: COURSE.id → courseId (ONE_TO_MANY) - "Get all enrollments for this course"

All four relationships must be wired after creating the junction table and before marking the task complete. The bidirectional pattern enables navigation from either direction. The implementation_notes from the planning phase will specify which entities the junction table connects.

### Bidirectional Relationships (Required for All FKs)

For EVERY foreign key field, wire TWO relationships to enable navigation in both directions. Appian stores relationships with record types, so a relationship from CASE to CASE_STATUS does not automatically create the inverse.

**One-to-Many Pattern:**

CASE has statusId (FK → CASE_STATUS)

Required relationships (both directions):
1. CASE.statusId → CASE_STATUS.id (MANY_TO_ONE) - "Get status for this case"
2. CASE_STATUS.id → CASE.statusId (ONE_TO_MANY) - "Get all cases with this status"

**Why Both Directions:**
- Query flexibility: Navigate parent→children OR children→parent
- Prevents runtime errors when traversing from "wrong" direction
- Essential for related record views, reports, and list filtering

Wire both directions immediately after creating the record type with the FK field. This applies to ALL foreign keys, not just junction tables.

### User-Type Fields → User System Record Type

Any field whose type is USER (e.g., assigned_to, submitted_by, reviewer) must have a MANY_TO_ONE relationship wired to the built-in User system record type after creation. Do not create a User entity — use the system record type instead.

- System record type UUID: `SYSTEM_RECORD_TYPE_USER`
- User primary key field UUID: `SYSTEM_RECORD_TYPE_USER_FIELD_username`
- The User record type is WEB_SERVICE-backed — you cannot insert data into it.

Name each relationship after the source field (e.g., a field called `assigned_to` gets a relationship named `assignedToUser`). Wire one relationship per USER-type field on the record type.

### Related Configuration

After creating the record type and wiring relationships, you may need to configure additional features. Check the plan's implementation_notes to determine which apply:

**User Filters** (see user_filter.md):
- Add if planning phase (record_types.md RULE 11) specifies filters
- Must be added AFTER creating record type but BEFORE inserting sample data
- Use addRecordTypeUserFilter with field UUIDs from creation response
- Typical sequence: Create record type → Add fields → Wire relationships → Add user filters → Insert sample data

**Sample Data** (see section below):
- Insert AFTER record type, relationships, and filters are configured
- Required for all database-backed record types

**Record Actions** (see record_action.md):
- Add AFTER sample data is inserted
- Requires interface and process model created separately (see interface.md, process_model.md)
- LIST_ACTION for creating new records, RELATED_ACTION for editing existing records

**Record Views** (when explicitly planned):
- Add AFTER sample data and actions (if planned)
- First view added becomes Summary view
- Subsequent views become tabs
- See planning_process.md line 134: "ONLY if the user explicitly requests summary/detail views"
- Use addRecordTypeView with interface expression
- (Future enhancement - detailed guidance to be added if views become commonly used)

**Record Events** (see section below):
- Configure AFTER all other setup complete
- Platform auto-creates supporting record types

### Sample Data

Every database-backed record type must have sample data inserted immediately after creation and relationship wiring, before marking the task complete. A record type without sample data is incomplete.

(Note: WEB_SERVICE and PROCESS-backed record types pull data from external sources and cannot have data inserted directly. This guide only covers database-backed types.)

**How many rows to generate**:

Different record type categories need different amounts of sample data.

**STANDARD volume** (default - supports dashboards and analytics):

- **Primary entities**: Insert 15-20 diverse, realistic examples
  - Supports meaningful charts, filters, and aggregations
  - Distribute across categories/statuses (not concentrated in one)
  - Span time periods (3-6 months, not single day)
  - Multiple items per user/assignee (2-4 each, not 1)
  - Example: 18 maintenance requests, 15 committees, 20 customers
  
- **Reference tables**: Insert ALL values specified in implementation_notes (typically 3-10 rows)
  - These are complete controlled vocabularies
  - Example: If implementation_notes says "Populate with: Low, Medium, High, Critical" → insert exactly 4 rows
  
- **Junction tables**: Insert 20-30 relationships demonstrating varied usage patterns
  - Show how entities relate in realistic ways
  - Demonstrate many-to-many patterns (e.g., members on multiple committees)
  - Sufficient for dashboard filtering and grouping
  - Example: 25 student-course enrollments, 28 member-committee relationships
  
- **Document entities**: Insert 8-10 structure examples
  - Show the structure without actual file content
  - Enough for document-based dashboards and searches
  - Example: 10 document records with realistic titles and metadata

**User can override volume**:
- User requests "minimal data", "quick prototype", "CRUD only" → 3-5 entities, 5-8 junction relationships
- User requests "rich data", "comprehensive sample data" → 40-50 entities, 60-80 junction relationships
- User requests "performance testing", "large dataset" → 100+ entities, 200+ junction relationships
- Note: Quality may become formulaic at very high volumes

**Data Distribution for Dashboards**:

When generating sample data for dashboard/analytics use cases:

1. **Distribute across categories**: If there are 5 statuses, ensure 3-4 rows per status (not all "Open")
2. **Spread across time**: Generate dates spanning 3-6 months (not all on the same day)
3. **Vary related fields**: Different priority + status combinations (not all "High + Open")
4. **Multiple per user**: If tracking assignments, give users 2-4 items each (not 1 each)
5. **Realistic value ranges**: For numeric fields, vary amounts ($50, $500, $5000 - not all $100)

**Example** (MAINTENANCE_REQUEST with 18 rows supporting dashboards):
- Status distribution: Open(4), In Progress(5), On Hold(2), Completed(5), Cancelled(2)
- Priority distribution: Low(4), Medium(7), High(5), Critical(2)
- Date spread: Spanning last 4 months (not all today)
- Assignment: 4 technicians with 3-6 requests each
- Cost range: $75 to $12,000 (realistic variety)

This distribution creates meaningful charts, useful filters, and realistic aggregations.

Use the field names from the creation response as CSV column headers — they must match exactly. Populate realistic, internally consistent data that demonstrates the record type's purpose.

**Generating realistic, domain-appropriate sample data**:

Use context from planning phase to generate high-quality sample data:

1. **Read the record type description** (from implementation_notes) to understand business purpose
2. **Read field descriptions** (from implementation_notes) to understand expected content
3. **Use sample data guidance** from implementation_notes (may be EXTRACTED from requirements or PROPOSED examples)
4. **Strip labeling markers** ([PROPOSED], [DERIVED], (Source: Requirements)) - these were for planning review only
5. **Generate domain-appropriate values** that match the guidance provided

**Quality Checklist**:
- ✅ Use domain-specific terminology (not generic "Item 1, Item 2")
- ✅ Values should be realistic and demonstrate business purpose
- ✅ Related fields should be consistent (e.g., title matches description)
- ✅ Dates should be realistic and ordered appropriately
- ✅ Varied data shows different scenarios (different statuses, priorities, etc.)
- ❌ Avoid sequential numbering (Committee 1, Committee 2, Committee 3)
- ❌ Avoid placeholder text (Test, TBD, TODO, Lorem ipsum)
- ❌ Avoid fabricated usernames (use real usernames from SYSTEM_RECORD_TYPE_USER)

**Example 1: COMMITTEE Record Type**

Implementation notes excerpt:
```
Description: Represents governance committees within the organization [PROPOSED - please verify]
Fields:
- name (TEXT, required)
- description (TEXT, optional)
- meetingFrequency (TEXT, optional) - How often the committee meets [PROPOSED - please verify]
- isActive (BOOLEAN, default=true)
Sample data: Generate 5 realistic governance committee examples (Finance Committee, Audit Committee, 
Technology Committee, Governance Committee, Compensation Committee) [PROPOSED - modify type/names as needed]
```

Note: The [PROPOSED] markers indicate these were suggested examples that user reviewed/approved during planning.

✅ GOOD CSV (domain-appropriate, demonstrates purpose):
```csv
name,description,meetingFrequency,isActive,createdAt,createdBy
Finance Committee,Oversees budget planning and financial compliance activities,Monthly,1,2026-01-15 09:00:00,john.smith
Audit Committee,Reviews internal controls and regulatory compliance,Quarterly,1,2026-01-15 09:00:00,john.smith
Technology Committee,Guides technology strategy and digital transformation initiatives,Monthly,1,2026-01-15 09:00:00,john.smith
Governance Committee,Ensures board effectiveness and corporate governance standards,Quarterly,1,2026-01-15 09:00:00,john.smith
Compensation Committee,Reviews executive compensation and benefits packages,Bi-annually,1,2026-01-15 09:00:00,john.smith
```

❌ BAD CSV (generic, doesn't demonstrate purpose):
```csv
name,description,meetingFrequency,isActive,createdAt,createdBy
Committee 1,Description for committee 1,Frequency 1,1,2026-01-15 09:00:00,john.smith
Committee 2,Description for committee 2,Frequency 2,1,2026-01-15 09:00:00,john.smith
Test Committee,Test description,TBD,1,2026-01-15 09:00:00,john.smith
```

**Note on usernames in examples:** The CSV examples above use placeholder usernames (john.smith, jane.martinez) for illustration purposes only. In actual execution, query SYSTEM_RECORD_TYPE_USER to get real usernames from the Appian directory before inserting sample data.

**Example 2: MAINTENANCE_REQUEST Record Type**

Implementation notes excerpt:
```
Description: Tracks equipment maintenance requests submitted by facilities team members 
(Source: Requirements - "track facility maintenance requests")
Fields:
- title (TEXT, required)
- description (TEXT, optional)
- statusId (INTEGER FK → REQUEST_STATUS)
- priorityId (INTEGER FK → REQUEST_PRIORITY)
Sample data: Generate 4 maintenance request examples with domain-appropriate titles 
(HVAC System Failure, Electrical Outlet Repair, Water Leak) demonstrating varied priorities 
and statuses [PROPOSED scenarios - modify as needed]
```

Note: Description was EXTRACTED from requirements; sample data examples were PROPOSED (user reviewed/approved).

✅ GOOD CSV (realistic scenarios):
```csv
title,description,statusId,priorityId,createdAt,createdBy
HVAC System Failure - Building A,Air conditioning not working in conference rooms on floor 3,1,4,2026-06-01 08:15:00,jane.martinez
Electrical Outlet Repair - Floor 2,Non-functional outlets in office 2-201 preventing equipment use,2,3,2026-06-01 10:30:00,mike.chen
Water Leak - Warehouse,Small pipe leak in storage area near loading dock,2,2,2026-06-01 13:45:00,sarah.jones
Lighting Replacement - Parking Lot,Several light fixtures out in employee parking area,1,1,2026-06-02 07:20:00,david.kim
```

❌ BAD CSV (generic):
```csv
title,description,statusId,priorityId,createdAt,createdBy
Request 1,Description for request 1,1,1,2026-06-01 08:15:00,user1
Maintenance Task 2,Some issue that needs fixing,2,2,2026-06-01 10:30:00,user2
Test Request,Test description,1,1,2026-06-01 13:45:00,testuser
```

**Example 3: Reference Table (REQUEST_STATUS)**

Implementation notes excerpt:
```
Description: Reference table containing valid statuses that control the lifecycle [DERIVED from common pattern]
Sample data: Populate with: Submitted, Assigned, In Progress, On Hold, Completed, Cancelled 
[PROPOSED status values - modify as needed]
```

Note: After user review during planning, these become the definitive values to insert.

✅ GOOD CSV (exactly as specified):
```csv
label,sortOrder,isActive
Submitted,1,1
Assigned,2,1
In Progress,3,1
On Hold,4,1
Completed,5,1
Cancelled,6,1
```

Reference tables are straightforward — use exactly the values specified in implementation_notes.

**Example 4: Junction Table (COMMITTEE_MEMBER)**

Implementation notes excerpt:
```
Description: Links members to committees with their roles and term dates [DERIVED from junction pattern]
Fields:
- id (INTEGER PK)
- committeeId (INTEGER FK → COMMITTEE)
- userId (USER FK → SYSTEM_RECORD_TYPE_USER)
- roleId (INTEGER FK → MEMBER_ROLE)
- startDate (DATE, required)
- endDate (DATE, optional) - Term end date; null indicates active membership
- isVotingMember (BOOLEAN, default=true)
Sample data: Generate 25 diverse member-committee relationships showing members serving on 
multiple committees with different roles (Chair, Member, Secretary) [PROPOSED - adjust as needed]
```

Note: Junction tables demonstrate many-to-many patterns and need sufficient data for dashboard filtering.

✅ GOOD CSV (demonstrates many-to-many patterns, variety, realistic distribution):
```csv
committeeId,userId,roleId,startDate,endDate,isVotingMember
1,john.smith,1,2025-01-01,,1
1,jane.martinez,2,2025-01-01,,1
1,david.kim,2,2025-01-01,,1
1,sarah.jones,2,2025-01-01,,0
2,john.smith,2,2025-03-15,,1
2,mike.chen,1,2025-03-15,,1
2,sarah.jones,2,2025-03-15,,1
2,emily.wong,2,2025-03-15,,1
3,jane.martinez,1,2025-06-01,,1
3,david.kim,2,2025-06-01,,1
3,mike.chen,2,2025-06-01,,0
3,robert.lee,2,2025-06-01,,1
4,mike.chen,3,2025-02-01,,1
4,sarah.jones,1,2025-02-01,,1
4,emily.wong,2,2025-02-01,,1
5,david.kim,1,2024-01-01,2024-12-31,1
5,robert.lee,2,2024-01-01,2024-12-31,1
1,michael.park,2,2024-06-01,2025-05-31,1
2,lisa.chen,2,2024-09-01,2025-08-31,1
3,james.wong,2,2025-01-01,,1
4,anna.kim,2,2025-01-01,,1
1,thomas.lee,2,2025-02-20,,1
2,jessica.park,2,2025-02-20,,1
3,william.chen,2,2025-03-10,,1
4,sophia.kim,2,2025-04-05,,1
```

Key features demonstrated:
- **Many-to-many**: john.smith on committees 1 & 2; mike.chen on 2, 3, 4
- **Role variety**: Chair (roleId=1), Member (roleId=2), Secretary (roleId=3)
- **Term dates**: Some active (endDate=null), some completed (endDate populated)
- **Voting status**: Mix of voting and non-voting members
- **Distribution**: 25 relationships across 5 committees (realistic spread)
- **Time spread**: Dates spanning 18 months (supports time-based dashboards)

❌ BAD CSV (doesn't demonstrate patterns, poor distribution):
```csv
committeeId,userId,roleId,startDate,endDate,isVotingMember
1,user1,1,2025-01-01,,1
2,user2,2,2025-01-01,,1
3,user3,3,2025-01-01,,1
4,user4,1,2025-01-01,,1
5,user5,2,2025-01-01,,1
```

Problems:
- Each user only on one committee (doesn't show many-to-many)
- All same date (no time distribution)
- No variety in membership patterns
- Only 5 rows (insufficient for dashboard filtering)
- Sequential assignment (unrealistic)

**CSV format rules:**
- Boolean: use 1 (true) or 0 (false) — NOT true/false strings
- Date: YYYY-MM-DD (e.g. 2026-03-20)
- Datetime: YYYY-MM-DD HH:MM:SS (e.g. 2026-03-20 14:30:00)
- Time: HH:MM:SS (e.g. 14:30:00)
- Integer FK fields (foreign keys to other tables): use primary keys from prior insert responses
- USER fields: query `SYSTEM_RECORD_TYPE_USER` for real usernames — do not fabricate
- Quote fields containing commas, quotes, or newlines per RFC 4180
- No JSON in CSV fields — plain text only
- Omit the primary key column if it is auto-generated

### Record Event Configuration

When the plan calls for audit tracking or event history on a record type, configure record events on the primary record type after creation. Use meaningful past-tense business event types (e.g., "Submitted", "Approved", "Rejected") — not generic labels.

Supporting record types (Event History, Event Type Lookup, Reply Thread, and Subscriber record types with proper relationships and security inherited from the base record type) are auto-created by the platform. Do not create these manually.

### Editing Existing Fields

When changing a field's type, only certain conversions are allowed:

- **Interchangeable**: TEXT ↔ USER ↔ EXTRA_LONG_TEXT
- **Interchangeable**: INTEGER ↔ GROUP ↔ DOCUMENT ↔ FOLDER
- **No type change supported**: BOOLEAN, DATE, DATETIME, TIME, DECIMAL, NUMBER

Note: To create a Long Text field (descriptions, notes, comments), use fieldType: TEXT with length: 4000. Use EXTRA_LONG_TEXT for content over 4000 characters. For editing existing fields, TEXT, USER, and EXTRA_LONG_TEXT types can be converted to each other.

Incompatible type changes are rejected by the API with HTTP 400.

If a user requests an incompatible type change, explain that the requested conversion is not supported. Suggest deleting the field and creating a new one with the desired type — note that existing data will be lost. Do not mention database columns, storage types, or internal implementation details in your response.

### Record Title Expression

After creating a record type and its fields, set the title expression so records display meaningful names instead of IDs. The title expression determines what text appears in record links, views, and headers.

**Format:** The title expression is a SAIL expression using `rv!record` to access field values. The field path must be wrapped in single quotes:

```
rv!record['recordType!{<record-type-uuid>}<RecordTypeName>.fields.{<field-uuid>}<fieldName>']
```

**Rules:**
- Set the title to the first meaningful text field (e.g., name, title, subject)
- For concatenated titles: `rv!record['recordType!{uuid}MyRecord.fields.{fieldUuid1}firstName'] & " " & rv!record['recordType!{uuid}MyRecord.fields.{fieldUuid2}lastName']`
- Use `getRecordType` to discover the current title expression, record type name, and field UUIDs only if the record type was created outside the current plan. For record types created in this plan, use the UUIDs from the "OBJECTS CREATED SO FAR" section.
- Call `updateRecordType` with `titleExpression` to set it
- The record type name and field name MUST match exactly what was returned by `getRecordType`

**When to set:** After creating a record type with its fields, always set the title expression to the primary text field. If no text field exists (e.g., a lookup table with only an ID and a name), use the name field.

**Example flow:**
1. Create record type "RE1 Employee" → get back UUID `aeeb7f1f-a599-47f0-818a-09ad83f42b6e`
2. The creation response includes fields with UUIDs, e.g., field `employeeName` has UUID `6acacae9-c909-4685-97a2-2b8aa511ba48`
3. Call `updateRecordType` with:
   ```
   titleExpression: "rv!record['recordType!{aeeb7f1f-a599-47f0-818a-09ad83f42b6e}RE1 Employee.fields.{6acacae9-c909-4685-97a2-2b8aa511ba48}employeeName']"
   ```
