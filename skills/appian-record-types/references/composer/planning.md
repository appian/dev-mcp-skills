## Appian Record Types — Platform-Specific Rules

These rules apply to Appian Record Types as design objects. They complement the generic data modeling principles.

╔══════════════════════════════════════════════════════════════════════════════╗
║  SYSTEM_RECORD_TYPE_USER — NEVER create entities for Appian login users.               ║
║  (USER, PARTNER, EMPLOYEE, AGENT, ASSIGNEE, MANAGER, STAFF, etc.)           ║
║  Use fields of type USER on transaction entities instead:                    ║
║    INSPECTION.assigned_to (type: USER) → SYSTEM_RECORD_TYPE_USER                        ║
║  Only create a person entity for EXTERNAL contacts with no Appian login.     ║
║  Use Appian Groups for role-based access — not role entities.                ║
╚══════════════════════════════════════════════════════════════════════════════╝

### SYSTEM_RECORD_TYPE_USER References (RULE 6)

Any field referencing an Appian user (submitted_by, assigned_to, created_by, reviewed_by, approved_by, manager) must have its field type set to USER and link to SYSTEM_RECORD_TYPE_USER.

**Field Type Selection:**
- Person with Appian login → USER type
- External contact (no Appian login) → TEXT fields (contactName, contactEmail)

**Pattern 1 - Profile Extension:**
When business entity extends an Appian user (Employee, Partner, Agent):
EMPLOYEE:
├─ id (INTEGER PK)
├─ appianUser (USER FK → SYSTEM_RECORD_TYPE_USER)      ← Link to system user
├─ employeeNumber (TEXT)
├─ departmentId (INTEGER FK)
└─ ...

**Pattern 2 - Audit/Tracking Fields:**
MAINTENANCE_REQUEST:
├─ id (INTEGER PK)
├─ createdBy (USER FK → SYSTEM_RECORD_TYPE_USER)       ← Audit field
├─ assignedTo (USER FK → SYSTEM_RECORD_TYPE_USER)      ← Workflow field
└─ ...

**Pattern 3 - External Contact (NOT a USER field):**
VENDOR_CONTACT:
├─ id (INTEGER PK)
├─ contactName (TEXT)                        ← Text field, not USER
├─ contactEmail (TEXT)
└─ vendorId (INTEGER FK)

State in implementation_notes which fields have type USER and that each needs a relationship to the User system record type (UUID: SYSTEM_RECORD_TYPE_USER). The execution agent will wire a MANY_TO_ONE relationship from each USER-type field to the built-in User system record type.

### Record Events (RULE 9)

If requirements mention "who submitted", "when modified", "audit trail", or "submission record" → plan record event configuration on the primary record type. Do NOT create a standalone audit/history entity.

### User Filters (RULE 11)

When requirements mention filtering, searching, narrowing, or browsing records by specific fields, filters MUST be applied directly to the Record Type as User Filters — NOT as interface-level filtering logic. This is the default unless the user EXPLICITLY requests interface-based filtering instead. Note filters in the Record Type's implementation_notes OR as a separate plan step.

Map to facet types:
- FK field (INTEGER) with a M:1 relationship → LIST_OF_VALUES (useRelatedRecordValues)
- Date/Datetime fields → DATE_RANGE
- Text fields (status, category, department, etc.) → EXPRESSION (use a!recordFilterList with aggregation to get distinct values)
- Boolean fields → LIST_OF_VALUES (explicit Yes/No options)
- Any field needing dynamic/computed options → EXPRESSION

### Filter Prerequisite Check (RULE 11a)

When planning a dashboard interface that uses filters, you MUST verify during discovery whether the target record type already has User Filters configured. If the record type does NOT have User Filters, you MUST include a plan step to configure User Filters on the record type BEFORE the dashboard interface step. Never plan a dashboard that relies on filters without ensuring the underlying record type has them.

────────────────────────────────────────────────────────────────────────────────
RECORD TYPE STEP FORMAT
────────────────────────────────────────────────────────────────────────────────

For each Record Type plan step, implementation_notes MUST include:
1. Whether it is Reference, Primary, Junction, or Document
2. Source traceability (Requirements quote OR Best Practice reasoning)
3. ALL fields the record type should have. For each primary entity, consider fields across these categories:
   - Descriptive (names, titles, descriptions)
   - Temporal (dates, timestamps, deadlines)
   - Quantitative (amounts, costs, counts, durations)
   - Contact (email, phone, address)
   - Classification (FK to each reference table)
   - User references (submittedBy, assignedTo, approvedBy)
   - Audit fields (createdAt, createdBy, modifiedAt, modifiedBy) - See data_model.md RULE 16
   - Soft delete fields (if required) - See data_model.md RULE 17
   - Free-text (notes, comments, reason)
   The execution agent relies on this list to create the record type correctly.
   
   **Field descriptions**: When requirements provide specific details about field formats, business rules, 
   or constraints, include those as field descriptions (see item 7 below for format and examples).

4. Fields of type USER that link to SYSTEM_RECORD_TYPE_USER (if any) — specify each field name, confirm its type is USER, and note that a relationship to the User system record type is required
5. Sample/initial data to populate: Specify what sample data to generate. Format depends on record type category.

   **Content Strategy** (consistent with descriptions):
   - **EXTRACTED**: Use specific values mentioned in user requirements
   - **PROPOSED**: Suggest realistic examples when requirements lack specifics (mark with [PROPOSED])
   - Use conservative, mainstream examples that user can review and modify
   
   **Reference tables**: List specific values from requirements or domain knowledge
   - EXTRACTED example: "Populate with: Draft, Under Review, Approved, Rejected, Archived (Source: Requirements)"
   - PROPOSED example: "Populate with: Low, Medium, High, Critical [PROPOSED priority levels - modify as needed]"
   - PROPOSED example: "Populate with: Electrical, Plumbing, HVAC, Structural, Safety, General Maintenance [PROPOSED categories - modify as needed]"
   
   **Primary entities**: Specify quantity and provide domain-appropriate guidance with distribution details
   - EXTRACTED example: "Generate 18 committee examples including Finance, Audit, Technology (from requirements); add similar governance committees [PROPOSED additional examples]. Distribute across active/inactive status."
   - PROPOSED example: "Generate 15 realistic governance committee examples (Finance Committee, Audit Committee, Technology Committee, Governance Committee, Compensation Committee, and 10 similar) [PROPOSED - modify type/names as needed]. Ensure variety in meeting frequency and creation dates spanning last 6 months."
   - PROPOSED example: "Generate 18 maintenance request examples with domain-appropriate titles (HVAC System Failure, Electrical Outlet Repair, Water Leak, and 15 similar realistic scenarios) [PROPOSED scenarios - modify as needed]. Distribute across all priority and status combinations, span last 4 months, assign 3-5 requests per technician."
   
   **Junction tables**: Specify relationships to demonstrate with enough variety for filtering
   - Example: "Generate 25 diverse member-committee relationships showing members serving on multiple committees with different roles (Chair, Member, Secretary) [PROPOSED - adjust as needed]. Ensure some members are on 2-3 committees and some terms have end dates."
   - Example: "Generate 28 student-course enrollments demonstrating students taking multiple courses across different semesters [PROPOSED - adjust as needed]. Distribute across 2-3 semesters, vary enrollment dates."
   
   **Quantity guidelines** (unless user specifies otherwise):
   - Reference tables: All specified values (typically 3-10 complete vocabulary)
   - Primary entities: 15-20 diverse, realistic examples (supports dashboards and analytics)
   - Junction tables: 20-30 relationships showing varied usage patterns
   - Rationale: Default volume supports charts, filters, and aggregations in dashboard interfaces
   
   For primary entities, provide enough variety to support dashboard use cases:
   - Distribute across status/category values (3-4 per category, not all in one)
   - Span time periods (e.g., "spanning last 3-6 months", not all same day)
   - Multiple per user/assignee (e.g., "2-4 requests per technician")
   - Realistic value ranges for numeric fields (e.g., "costs ranging $50-$5000")
   
   User can request volume adjustments:
   - "minimal data" or "quick prototype" → 3-5 entities, 5-8 junction relationships
   - "rich data" or "comprehensive sample data" → 40-50 entities, 60-80 junction relationships
   
   **Quality Guidelines for Sample Data Proposals**:
   - ✅ Propose realistic, mainstream examples users can relate to
   - ✅ Use common domain terminology (governance committees, maintenance types)
   - ✅ Show variety (different statuses, priorities, scenarios)
   - ❌ Don't invent organization-specific names/codes
   - ❌ Don't use numbered placeholders (Committee 1, Request 2)
   - User reviews during planning phase and can modify examples
   
   This MUST be part of the record type's implementation_notes — NEVER a separate plan step. A standalone "Populate Sample Data" step is ONLY valid when adding data to an EXISTING record type.

6. Record type description: A clear 1-2 sentence explanation of what this record type represents and its business purpose. This description will:
   - Appear in Appian Designer to help developers understand the record type
   - Guide the execution agent in generating contextually appropriate sample data
   - Serve as documentation for future maintenance
   
   **Content Strategy**:
   - **EXTRACTED** from requirements → `(Source: Requirements - "quote")` - Direct quotes or paraphrases from user's request
   - **PROPOSED** when details lacking → `[PROPOSED]` - Reasonable defaults based on mainstream patterns, user reviews during planning
   - **DERIVED** from patterns → `[DERIVED]` - Logical inference from common patterns
   
   **Quality Guidelines for Proposals**:
   - ✅ Use generic, widely applicable descriptions
   - ✅ Reference entity/process mentioned in user request
   - ✅ Use common domain terminology (governance, maintenance, service)
   - ❌ Don't invent organization-specific details
   - ❌ Don't propose niche or overly specific terminology
   - ❌ Don't fabricate business rules not mentioned in requirements
   
   **Good vs. Bad PROPOSED Examples**:
   
   User request: "Create an app to track committees"
   
   ✅ GOOD PROPOSED (generic, widely applicable):
   ```
   Description: Represents governance committees within the organization [PROPOSED]
   ```
   
   ❌ BAD PROPOSED (too specific, invented org structure):
   ```
   Description: Represents Board of Directors committees including Finance (CFO-chaired), 
   Audit (external auditor reporting), and Compensation (CEO compensation oversight) per 
   corporate bylaws section 3.2 [PROPOSED]
   ```
   Problems: Invented org structure, specific roles, reporting lines, policy references
   
   **Key principle**: Propose mainstream defaults that user can refine, not organization-specific business logic.

7. Field descriptions (for non-obvious fields): When requirements specify formats, constraints, or business rules 
   for a field, capture those as field descriptions. These descriptions will:
   - Appear in Appian Designer when developers configure forms and grids
   - Help the execution agent understand expected field content for sample data
   - Document business rules and constraints provided by the user
   
   **When to add field descriptions**:
   - ✅ User specifies field format or constraints (e.g., "code format is XXX-000-000", "max 50 characters")
   - ✅ User specifies business rules (e.g., "expires on this date", "required for approval")
   - ✅ Business-specific flags with non-obvious behavior
   - ✅ Status/state fields that control workflow
   - ❌ Self-explanatory fields: name, email, phone, createdAt (don't need descriptions)
   
   **Priority**: If user requirements provide specific details about a field, ALWAYS include those as the 
   field description. This is particularly important for code/identifier fields, constrained values, and 
   fields with validation rules.
   
   **Content Strategy** (same as record type descriptions):
   - **EXTRACTED** from requirements → `(Source: Requirements - "quote")`
   - **PROPOSED** when lacking detail → `[PROPOSED]`
   - **DERIVED** from patterns → `[DERIVED]`
   
   **Quality Guidelines for Proposals**:
   - ✅ Use mainstream interpretations of common field names
   - ✅ Base on common domain patterns (isActive = visibility, not deletion)
   - ✅ Keep descriptions brief (one sentence)
   - ❌ Don't invent organization-specific business rules
   - ❌ Don't propose overly specific behaviors
   - ❌ Don't fabricate technical constraints not in requirements
   
   **Good vs. Bad PROPOSED Field Descriptions**:
   
   User request: "Add isActive field to committee table"
   
   ✅ GOOD PROPOSED (common pattern, simple):
   ```
   isActive (BOOLEAN) - Whether the committee is currently active and appears in dropdowns [PROPOSED]
   ```
   
   ❌ BAD PROPOSED (invented business rules):
   ```
   isActive (BOOLEAN) - Controls committee status. When false, committee is deactivated 
   and only users with 'Admin' or 'Committee Manager' roles can view. Automatically set 
   to false after 12 months of inactivity per retention policy. [PROPOSED]
   ```
   Problems: Invented role-based access, auto-deactivation logic, retention policies
   
   **Format**: Include field descriptions inline in the field list using this pattern:
   ```
   fieldName (TYPE, required/optional) - Description text [PROPOSED/DERIVED] or (Source: Requirements - "quote")
   ```
   
   Examples:
   - With user-provided details: `accountNumber (TEXT, required) - Must be exactly 10 digits (Source: Requirements - "account numbers are 10 digits")`
   - With proposed description: `isActive (BOOLEAN, required, default: true) - Controls whether item appears in active dropdowns [PROPOSED]`
   - Without description (self-explanatory): `name (TEXT, required)`
   
   The execution agent will extract descriptions (text after the hyphen) and pass them to the LCP tool.

### Record Title Expression (RULE 12)

Every record type should have a title expression set after creation. The title expression determines what text appears in record links, views, and headers. In implementation_notes, specify which field should be used as the title — typically the first meaningful text field (e.g., name, title, subject). The execution agent will set the title expression using `updateRecordType` after the record type and its fields are created.

For reference/lookup tables: use the name/label field.
For primary entities: use the most identifying text field (e.g., requestTitle, employeeName, orderNumber).
