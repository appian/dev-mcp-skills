# Record Query Patterns

Comprehensive patterns for querying Appian record data, including when to use each query function, relationships, paging, sorting, filtering, and aggregations.

**Related references:**
- [function-reference.md](function-reference.md) - Function signatures and quick examples
- [null-safety-patterns.md](null-safety-patterns.md) - Null handling patterns
- [array-patterns.md](array-patterns.md) - Array manipulation patterns

---

## Choosing the Right Query Function

**CRITICAL:** Use the right function for your use case to avoid performance issues and hit correct limits.

| Use Case | Function | Returns | Max Related Records |
|----------|----------|---------|---------------------|
| **Query multiple records** with filters, sorting | `a!queryRecordType()` | Array of records | 100 per base record |
| **Load ONE specific record** by identifier/PK | `a!queryRecordByIdentifier()` | Single record | 250 per relationship |
| **Grid/chart data source** (component parameter only) | `a!recordData()` | Data source object | 100 per base record |
| **Aggregate data** (COUNT, SUM, AVG, etc.) | `a!queryRecordType()` with `a!aggregationFields()` | Array of maps | N/A (no related records) |

### Decision Tree

```
Need to query records?
├─ Know the specific record identifier/PK?
│  └─ YES → Use a!queryRecordByIdentifier()
│     - Single record by ID
│     - Summary views, record actions
│     - After Write Records in process
│     - Need >100 related records
│
├─ Need aggregations (COUNT, SUM, AVG)?
│  └─ YES → Use a!queryRecordType() with a!aggregationFields()
│     - KPIs, charts, metrics
│     - Cannot use with relatedRecordData
│
├─ Querying for grid/chart component?
│  └─ YES → Use a!recordData() in component's data parameter
│     - NEVER store in local! variable
│     - Only for component parameters
│
└─ Otherwise → Use a!queryRecordType()
   - Multiple records with filters
   - Search results
   - Lists with paging
```

**Common mistake:**
```sail
/* ❌ WRONG - Using a!queryRecordType() when you have a specific ID */
local!case: a!queryRecordType(
  recordType: 'recordType!Case',
  filters: a!queryFilter(field: 'recordType!Case.fields.id', operator: "=", value: ri!caseId),
  fields: {...},
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 1),
  fetchTotalCount: true
).data[1]  /* Returns array, need [1] to get record */

/* ✅ CORRECT - Use a!queryRecordByIdentifier() for single record by ID */
local!case: a!queryRecordByIdentifier(
  recordType: 'recordType!Case',
  identifier: ri!caseId,
  fields: {...}
)  /* Returns single record directly, no [1] needed */
```

---

## a!queryRecordByIdentifier() - Single Record Queries

**Use for:**
- Summary views showing one specific record
- Record action interfaces (after Write Records in process)
- Detail pages when you have the record ID/identifier
- When you need >100 related records (max 250)

### Basic Pattern

```sail
local!case: a!queryRecordByIdentifier(
  recordType: 'recordType!Case',
  identifier: ri!caseId,  /* Primary key value */
  fields: {
    'recordType!Case.fields.id',
    'recordType!Case.fields.title',
    'recordType!Case.fields.description',
    'recordType!Case.fields.status'
  }
)
```

**Return type:** Single record (NOT an array) — access fields directly without `[1]`

### Single vs Composite Primary Key

**Identifying the PK type:**

When you call `getRecordType()`, check the `isPrimaryKey` field on each field:
- **Single PK**: One field has `"isPrimaryKey": true` → Pass the value directly
- **Composite PK**: Multiple fields have `"isPrimaryKey": true` → Use record constructor syntax

**Single Primary Key (most common):**
```sail
/* getRecordType shows: {"fieldName": "id", "isPrimaryKey": true} */

local!case: a!queryRecordByIdentifier(
  recordType: 'recordType!Case',
  identifier: ri!caseId,  /* Just pass the value directly */
  fields: {...}
)
```

**Composite Primary Key (less common):**
```sail
/* getRecordType shows multiple fields with "isPrimaryKey": true:
   {"fieldName": "partId", "isPrimaryKey": true}
   {"fieldName": "supplierId", "isPrimaryKey": true}
*/

local!partSupplier: a!queryRecordByIdentifier(
  recordType: 'recordType!Part Supplier',
  identifier: 'recordType!Part Supplier'(  /* Record type as constructor */
    'recordType!Part Supplier.fields.partId': 4,    /* All PK fields REQUIRED */
    'recordType!Part Supplier.fields.supplierId': 45
  ),
  fields: {...}
)
```

**CRITICAL: Record constructor syntax** - `'recordType!Type'(field: value, field: value)`
- Uses record type reference as a constructor (like a function call)
- Must include **all** primary key fields
- Must use **full UUID-qualified field references**
- Parentheses after record type, NOT curly braces

**Common mistakes with composite PK:**
```sail
/* ❌ WRONG - Plain map doesn't work */
identifier: {
  partId: 4,
  supplierId: 45
}

/* ❌ WRONG - a!map doesn't work */
identifier: a!map(
  partId: 4,
  supplierId: 45
)

/* ❌ WRONG - Short field names don't work */
identifier: 'recordType!Part Supplier'(
  partId: 4,          /* Missing full field reference! */
  supplierId: 45
)

/* ✅ CORRECT - Record constructor with full field references */
identifier: 'recordType!Part Supplier'(
  'recordType!Part Supplier.fields.partId': 4,
  'recordType!Part Supplier.fields.supplierId': 45
)
```

### With Related Records (1:M)

**Max 250 related records** (vs 100 for `a!queryRecordType()`):

```sail
local!customer: a!queryRecordByIdentifier(
  recordType: 'recordType!Customer',
  identifier: ri!customerId,
  fields: {
    'recordType!Customer.fields.fullName',
    'recordType!Customer.fields.email',
    /* Related case fields */
    'recordType!Customer.relationships.cases.fields.id',
    'recordType!Customer.relationships.cases.fields.title',
    'recordType!Customer.relationships.cases.fields.status'
  },
  relatedRecordData: {
    a!relatedRecordData(
      relationship: 'recordType!Customer.relationships.cases',
      limit: 250,  /* Up to 250, vs 100 max for a!queryRecordType() */
      sort: a!sortInfo(
        field: 'recordType!Case.fields.createdAt',
        ascending: false
      )
    )
  }
)
```

### Fields Parameter Behavior

**Omitting fields returns ONLY primary key:**
```sail
/* ❌ PROBLEM - Only PK returned, other fields null */
local!case: a!queryRecordByIdentifier(
  recordType: 'recordType!Case',
  identifier: ri!caseId
  /* Missing fields parameter! */
)
/* Result: only 'recordType!Case.fields.id' is populated */

/* ✅ CORRECT - Specify all fields needed */
local!case: a!queryRecordByIdentifier(
  recordType: 'recordType!Case',
  identifier: ri!caseId,
  fields: {
    'recordType!Case.fields.id',
    'recordType!Case.fields.title',
    'recordType!Case.fields.status'
  }
)
```

### Restrictions

**Cannot be used for:**
- ❌ Aggregations (`a!aggregationFields()` not supported)
- ❌ Looping over multiple identifiers (use `a!queryRecordType()` instead)
- ❌ Portals
- ❌ When you need to filter/search (no filters parameter)

**Anti-pattern:**
```sail
/* ❌ WRONG - Looping with a!queryRecordByIdentifier() */
a!forEach(
  items: ri!caseIds,
  expression: a!queryRecordByIdentifier(
    recordType: 'recordType!Case',
    identifier: fv!item,
    fields: {...}
  )
)

/* ✅ CORRECT - Use a!queryRecordType() with filters */
a!queryRecordType(
  recordType: 'recordType!Case',
  filters: a!queryFilter(
    field: 'recordType!Case.fields.id',
    operator: "in",
    value: ri!caseIds
  ),
  fields: {...},
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

### When You Need >100 Related Records

**If you need more than 100 related records:**
```sail
/* Option 1: Use a!queryRecordByIdentifier (max 250) */
local!customer: a!queryRecordByIdentifier(
  recordType: 'recordType!Customer',
  identifier: ri!customerId,
  fields: {...},
  relatedRecordData: {
    a!relatedRecordData(
      relationship: 'recordType!Customer.relationships.cases',
      limit: 250
    )
  }
)

/* Option 2: Query related record type separately (max unlimited with paging) */
local!customer: a!queryRecordByIdentifier(
  recordType: 'recordType!Customer',
  identifier: ri!customerId,
  fields: {...}  /* No related records */
),
local!cases: a!queryRecordType(
  recordType: 'recordType!Case',
  filters: a!queryFilter(
    field: 'recordType!Case.fields.customerId',
    operator: "=",
    value: ri!customerId
  ),
  fields: {...},
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 1000),
  fetchTotalCount: true
).data
```

---

## a!queryRecordType() - Multiple Record Queries

## CRITICAL: Required Parameters

Every `a!queryRecordType()` MUST include:

```sail
a!queryRecordType(
  recordType: 'recordType!Type',
  fields: { /* ALL fields you need */ },              /* REQUIRED - omitting returns only primary key */
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: N), /* REQUIRED */
  fetchTotalCount: true                               /* REQUIRED - enables .totalCount */
)
```

**Why these are mandatory:**
- **fields**: Without it, only the primary key is populated; all other fields return null
- **pagingInfo**: Always required, even for single-record queries
- **fetchTotalCount**: Enables `.totalCount` property for pagination UI

---

## Basic Query Patterns

### Simple Query with Fields

```sail
local!cases: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: {
    'recordType!Case.fields.id',
    'recordType!Case.fields.title',
    'recordType!Case.fields.status',
    'recordType!Case.fields.createdAt'
  },
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

**Returns:** Array of record instances (typed objects)

**Access fields:** Use full field references
```sail
local!firstTitle: local!cases[1]['recordType!Case.fields.title']
```

### Query with Filter

```sail
local!openCases: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: {
    'recordType!Case.fields.id',
    'recordType!Case.fields.title'
  },
  filters: a!queryFilter(
    field: 'recordType!Case.fields.status',
    operator: "=",
    value: "Open"
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

### Query with Multiple Filters (AND Logic)

```sail
local!filtered: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: { /* fields list */ },
  filters: a!queryLogicalExpression(
    operator: "AND",
    filters: {
      a!queryFilter(
        field: 'recordType!Case.fields.status',
        operator: "=",
        value: "Open"
      ),
      a!queryFilter(
        field: 'recordType!Case.fields.priority',
        operator: "=",
        value: "High"
      )
    }
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

---

## Relationships in Queries

**Critical distinction:**
- **Many-to-One (M:1)**: Include fields directly in `fields` parameter
- **One-to-Many (1:M)**: Use `a!relatedRecordData()` in `relatedRecordData` parameter

### UUID Organization for Multiple Related Types

**CRITICAL:** When querying with multiple related record types, organize UUIDs systematically to prevent copy-paste errors.

**Discovery workflow:**
```
Step 1: Base record type
getRecordType(caseUuid)
  → Save: Case fields (id, title, description, assignedTo, statusId, priorityId, etc.)
  → Save: Relationship UUIDs (status: {rel-uuid-1}, priority: {rel-uuid-2})

Step 2: First related record type
getRecordType(statusUuid)
  → Save: Status fields (id: {status-id-uuid}, label: {status-label-uuid})
  
Step 3: Second related record type
getRecordType(priorityUuid)
  → Save: Priority fields (id: {priority-id-uuid}, label: {priority-label-uuid})
```

**Build field references carefully:**
```sail
/* ✅ CORRECT - Each path uses UUIDs from correct getRecordType call */
fields: {
  /* Base record fields - from getRecordType(Case) */
  'recordType!{case-uuid}Case.fields.{case-id-uuid}id',
  'recordType!{case-uuid}Case.fields.{case-title-uuid}title',
  
  /* Status relationship - uses Status relationship UUID + Status field UUIDs */
  'recordType!{case-uuid}Case.relationships.{status-rel-uuid}status.fields.{status-id-uuid}id',
  'recordType!{case-uuid}Case.relationships.{status-rel-uuid}status.fields.{status-label-uuid}label',
  
  /* Priority relationship - uses Priority relationship UUID + Priority field UUIDs */
  'recordType!{case-uuid}Case.relationships.{priority-rel-uuid}priority.fields.{priority-id-uuid}id',
  'recordType!{case-uuid}Case.relationships.{priority-rel-uuid}priority.fields.{priority-label-uuid}label'
}

/* ❌ WRONG - Using Status label UUID in Priority path */
'recordType!{case-uuid}Case.relationships.{priority-rel-uuid}priority.fields.{status-label-uuid}label'
/* This fails with: "The field [...] used in 'fields' was invalid" */
```

**Common UUID mix-up scenario:**

When multiple related record types have similar structures (e.g., Status and Priority both have `id` and `label` fields), it's easy to accidentally reuse a UUID from the wrong record type.

**Prevention checklist:**
- [ ] Relationship UUID came from base record type's `relationships` array
- [ ] Field UUIDs came from the correct related record type's `fields` array
- [ ] Status field UUIDs → Status paths only
- [ ] Priority field UUIDs → Priority paths only
- [ ] Never mix field UUIDs between different related record types

**Why this matters:**

UUIDs are unique per field per record type. Even if two record types have identically named fields (`label`), their field UUIDs are different. Using the wrong UUID causes validation errors like:
```
The field [recordType!Case.relationships.priority.fields.{wrong-uuid}label] used in "fields" was invalid
```

### Including Related Fields (Many-to-One)

**Pattern:** Include specific fields from the related record type using the relationship path.

**Common use case:** Getting `id` and `label` from lookup tables for use in dropdowns or display.

```sail
local!cases: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: {
    /* Case fields */
    'recordType!Case.fields.id',
    'recordType!Case.fields.title',
    'recordType!Case.fields.statusId',
    
    /* Related status fields - specify individual fields */
    'recordType!Case.relationships.status.fields.id',      /* For component value */
    'recordType!Case.relationships.status.fields.label',   /* For display name */
    'recordType!Case.relationships.status.fields.color',   /* Additional display attribute */
    
    /* Related priority fields */
    'recordType!Case.relationships.priority.fields.id',
    'recordType!Case.relationships.priority.fields.label'
  },
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

**When to include the relationship itself:**
- Include `'recordType!Case.relationships.status'` (without `.fields.X`) **only if** you want ALL fields from the related record
- **Most common:** Include only specific fields like `.fields.id` and `.fields.label`

**Access related fields:**
```sail
/* Get status label from first case */
local!statusLabel: local!cases[1]['recordType!Case.relationships.status.fields.label']

/* In a!forEach */
a!forEach(
  items: local!cases,
  expression: fv!item['recordType!Case.relationships.priority.fields.label']
)
```

**Common pattern for dropdowns/display:**
```sail
/* Query returns id + label for related records */
fields: {
  'recordType!Case.fields.id',
  'recordType!Case.relationships.status.fields.id',     /* value */
  'recordType!Case.relationships.status.fields.label'   /* display */
}
```

### Including Related Records (One-to-Many)

**CRITICAL:** One-to-many relationships behave differently than many-to-one:

| Aspect | Many-to-One (M:1) | One-to-Many (1:M) |
|--------|-------------------|-------------------|
| **How to include** | Add to `fields` parameter | Use `a!relatedRecordData()` |
| **Default limit** | N/A (single record) | 10 records |
| **Max limit** | N/A | 100 records (with a!queryRecordType) |
| **Can sort/filter** | Sort base query, filter base query | Sort/filter within a!relatedRecordData() |
| **Example** | Case → Status (one status per case) | Case → Notes (many notes per case) |

**Without `a!relatedRecordData()` - Limited to 10:**
```sail
/* ❌ PROBLEM: Only returns first 10 notes per case (default limit) */
local!cases: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: {
    'recordType!Case.fields.id',
    'recordType!Case.fields.title',
    'recordType!Case.relationships.notes.fields.description',  /* Only first 10 notes! */
    'recordType!Case.relationships.notes.fields.createdAt'
  },
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

**With `a!relatedRecordData()` - Full control:**
```sail
/* ✅ CORRECT: Specify limit, sort, and filters for related notes */
local!cases: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: {
    'recordType!Case.fields.id',
    'recordType!Case.fields.title',
    'recordType!Case.relationships.notes.fields.description',
    'recordType!Case.relationships.notes.fields.createdAt'
  },
  relatedRecordData: {
    a!relatedRecordData(
      relationship: 'recordType!Case.relationships.notes',
      limit: 3,  /* Last 3 notes instead of default 10 */
      sort: a!sortInfo(
        field: 'recordType!Note.fields.createdAt',
        ascending: false  /* Newest first */
      )
    )
  },
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

**Pattern: Get latest related records with filters**
```sail
/* Get last 3 INTERNAL notes per case */
relatedRecordData: {
  a!relatedRecordData(
    relationship: 'recordType!Case.relationships.notes',
    filters: a!queryFilter(
      field: 'recordType!Note.fields.noteType',
      operator: "=",
      value: "Internal"
    ),
    sort: a!sortInfo(
      field: 'recordType!Note.fields.updatedAt',
      ascending: false
    ),
    limit: 3
  )
}
```

**Multiple related record sets:**
```sail
relatedRecordData: {
  /* Latest internal notes */
  a!relatedRecordData(
    relationship: 'recordType!Case.relationships.notes',
    filters: a!queryFilter(
      field: 'recordType!Note.fields.noteType',
      operator: "=",
      value: "Internal"
    ),
    sort: a!sortInfo(field: 'recordType!Note.fields.createdAt', ascending: false),
    limit: 3
  ),
  /* All attachments */
  a!relatedRecordData(
    relationship: 'recordType!Case.relationships.attachments',
    limit: 100
  )
}
```

**⚠️ CRITICAL: Field references in filters/sorts use the RELATED record type**
```sail
/* ✅ CORRECT - Field references start from Note, not Case */
a!relatedRecordData(
  relationship: 'recordType!Case.relationships.notes',
  filters: a!queryFilter(
    field: 'recordType!Note.fields.noteType',  /* Note fields, not Case fields */
    operator: "=",
    value: "Internal"
  )
)

/* ❌ WRONG - Don't use Case fields in related record filters */
a!relatedRecordData(
  relationship: 'recordType!Case.relationships.notes',
  filters: a!queryFilter(
    field: 'recordType!Case.fields.status',  /* ERROR - Case field not allowed here */
    operator: "=",
    value: "Open"
  )
)
```

**⚠️ Relationship type validation:**
```sail
/* ❌ WRONG - Using many-to-one relationship */
relatedRecordData: {
  a!relatedRecordData(
    relationship: 'recordType!Case.relationships.status'  /* This is M:1, not 1:M! */
  )
}
/* ERROR: "Only one-to-many type record type relationship references can be used" */

/* ✅ CORRECT - For many-to-one, add to fields parameter */
fields: {
  'recordType!Case.relationships.status.fields.label'  /* M:1 goes in fields */
}
```

**Key rules:**
1. **Default limit is 10** — always specify `limit` if you need more or fewer
2. **Max limit is 100** for `a!queryRecordType()` (250 for a!queryRecordByIdentifier)
3. **Only one-to-many relationships** — many-to-one must use `fields` parameter
4. **Limit applies per base record** — 3 notes per case, not 3 notes total
5. **Field references start from related record type** — not the base record type

### Filtering by Related Fields (Many-to-One)

```sail
local!highPriorityCases: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: { /* fields list */ },
  filters: a!queryFilter(
    field: 'recordType!Case.relationships.priority.fields.name',
    operator: "=",
    value: "High"
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

---

## Paging and Sorting

### Basic Paging

```sail
a!queryRecordType(
  recordType: 'recordType!Case',
  fields: { /* fields */ },
  pagingInfo: a!pagingInfo(
    startIndex: 1,
    batchSize: 50
  ),
  fetchTotalCount: true
)
```

**Batch size recommendations by use case:**
- **Dropdowns/pickers**: `batchSize: 100` (lookup tables, reference data)
- **Grid displays**: `batchSize: 20` (UI lists, case management)
- **Data export/processing**: `batchSize: 1000` (bulk operations)

**Default sorting:**
- Without explicit `sort`, results are sorted by the primary key of the base record type
- Add `sort` parameter in `pagingInfo` to override default ordering

**Access pagination info:**
```sail
local!query: a!queryRecordType(...),
local!data: local!query.data,           /* Array of records */
local!totalCount: local!query.totalCount, /* Total matching records */
local!batchSize: local!query.batchSize    /* Records per page */
```

### Sorting Results

**CRITICAL:** The parameter is `sort` (singular), NOT `sorts` (plural). It goes **inside pagingInfo**.

**Single sort field:**
```sail
a!queryRecordType(
  recordType: 'recordType!Case',
  fields: { /* fields */ },
  pagingInfo: a!pagingInfo(
    startIndex: 1,
    batchSize: 100,
    sort: a!sortInfo(
      field: 'recordType!Case.fields.createdAt',
      ascending: false  /* false = descending (newest first) */
    )
  ),
  fetchTotalCount: true
).data
```

**Multiple sort fields (array of a!sortInfo):**
```sail
pagingInfo: a!pagingInfo(
  startIndex: 1,
  batchSize: 100,
  sort: {
    a!sortInfo(field: 'recordType!Case.fields.priority', ascending: false),
    a!sortInfo(field: 'recordType!Case.fields.createdAt', ascending: false)
  }
)
```

**Pattern:**
- Single sort: `sort: a!sortInfo(...)`
- Multiple sorts: `sort: { a!sortInfo(...), a!sortInfo(...) }`

### Sorting by Related Fields

```sail
pagingInfo: a!pagingInfo(
  startIndex: 1,
  batchSize: 100,
  sort: {
    a!sortInfo(
      field: 'recordType!Case.relationships.status.fields.name',
      ascending: true
    )
  }
),
```

---

## Advanced Filtering

### Filters with Variables (Conditional Application)

**Rule:** Any filter whose `value` comes from a variable MUST include `applyWhen` to handle null/empty values.

```sail
a!queryRecordType(
  recordType: 'recordType!Case',
  fields: { /* fields */ },
  filters: a!queryLogicalExpression(
    operator: "AND",
    filters: {
      /* Always applied - literal value */
      a!queryFilter(
        field: 'recordType!Case.fields.isActive',
        operator: "=",
        value: true
      ),
      /* Conditionally applied - variable value */
      a!queryFilter(
        field: 'recordType!Case.fields.status',
        operator: "=",
        value: ri!selectedStatus,
        applyWhen: a!isNotNullOrEmpty(ri!selectedStatus)  /* REQUIRED for variables */
      ),
      a!queryFilter(
        field: 'recordType!Case.fields.assignedTo',
        operator: "=",
        value: local!selectedUser,
        applyWhen: a!isNotNullOrEmpty(local!selectedUser)
      )
    }
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

### Nested Logical Expressions (AND + OR)

**CRITICAL:** Nested `a!queryLogicalExpression()` must go in the `logicalExpressions` parameter, NOT in `filters`.

```sail
filters: a!queryLogicalExpression(
  operator: "AND",
  filters: {
    /* Direct filters at AND level */
    a!queryFilter(field: 'recordType!Case.fields.isActive', operator: "=", value: true)
  },
  logicalExpressions: {
    /* Nested OR expression */
    a!queryLogicalExpression(
      operator: "OR",
      filters: {
        a!queryFilter(field: 'recordType!Case.fields.priority', operator: "=", value: "High"),
        a!queryFilter(field: 'recordType!Case.fields.priority', operator: "=", value: "Critical")
      }
    )
  }
)

/* Result: isActive = true AND (priority = "High" OR priority = "Critical") */
```

### Common Filter Operators

| Data Type | Valid Operators |
|-----------|----------------|
| **Text** | `=`, `<>`, `in`, `not in`, `starts with`, `ends with`, `includes`, `not includes`, `is null`, `not null` |
| **Number** | `=`, `<>`, `>`, `>=`, `<`, `<=`, `between`, `in`, `not in`, `is null`, `not null` |
| **Date/DateTime** | `=`, `<>`, `>`, `>=`, `<`, `<=`, `between`, `in`, `not in`, `is null`, `not null` |
| **Boolean** | `=`, `<>`, `in`, `not in`, `is null`, `not null` |
| **User** | `=`, `<>`, `in`, `not in`, `is null`, `not null` |

---

## Aggregation Queries (KPIs and Metrics)

### Simple Count

```sail
local!totalCases: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: a!aggregationFields(
    measures: {
      a!measure(
        function: "COUNT",
        field: 'recordType!Case.fields.id',  /* field is REQUIRED even for COUNT */
        alias: "count"
      )
    }
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 1),
  fetchTotalCount: true
).data
```

**Extract value:**
```sail
local!count: if(
  a!isNotNullOrEmpty(local!totalCases),
  local!totalCases[1].count,  /* Use alias as property name */
  0
)
```

**Returns:** Array of **maps** (not record instances). Access using text alias.

### Count with Filter

```sail
local!openCasesQuery: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: a!aggregationFields(
    measures: {
      a!measure(function: "COUNT", field: 'recordType!Case.fields.id', alias: "count")
    }
  ),
  filters: a!queryFilter(
    field: 'recordType!Case.fields.status',
    operator: "=",
    value: "Open"
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 1),
  fetchTotalCount: true
).data,

local!openCount: if(
  a!isNotNullOrEmpty(local!openCasesQuery),
  local!openCasesQuery[1].count,
  0
)
```

### Grouped Aggregation (Multiple KPIs from One Query)

```sail
local!casesByStatus: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: a!aggregationFields(
    groupings: {
      a!grouping(
        field: 'recordType!Case.fields.status',
        alias: "status"
      )
    },
    measures: {
      a!measure(
        function: "COUNT",
        field: 'recordType!Case.fields.id',
        alias: "count"
      )
    }
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

**Returns:** Array of maps like:
```
{
  {status: "Open", count: 15},
  {status: "Closed", count: 23},
  {status: "In Progress", count: 8}
}
```

**Extract specific values:**
```sail
/* Get count for "Open" status */
local!openCount: if(
  a!isNotNullOrEmpty(local!casesByStatus),
  if(
    contains("Open", local!casesByStatus.status),
    local!casesByStatus.count[
      wherecontains("Open", local!casesByStatus.status)[1]
    ],
    0
  ),
  0
)

/* Or iterate through all groups */
a!forEach(
  items: local!casesByStatus,
  expression: fv!item.status & ": " & fv!item.count
)
```

### Other Aggregation Functions

```sail
fields: a!aggregationFields(
  groupings: { /* optional groupings */ },
  measures: {
    a!measure(function: "COUNT", field: 'recordType!...', alias: "total"),
    a!measure(function: "SUM", field: 'recordType!...fields.amount', alias: "totalAmount"),
    a!measure(function: "AVG", field: 'recordType!...fields.score', alias: "avgScore"),
    a!measure(function: "MIN", field: 'recordType!...fields.date', alias: "earliest"),
    a!measure(function: "MAX", field: 'recordType!...fields.date', alias: "latest")
  }
)
```

---

## Result Handling Patterns

### Checking for Empty Results

```sail
/* Always check before accessing .data */
local!query: a!queryRecordType(...),

local!isEmpty: a!isNullOrEmpty(local!query.data),

/* Safe property access */
local!firstRecord: if(
  not(local!isEmpty),
  local!query.data[1],
  null
)
```

### Property Access by Query Type

| Query Type | Returns | Property Access | Example |
|------------|---------|-----------------|---------|
| **Regular fields query** | Record instances (typed) | Full field references | `item['recordType!Type.fields.name']` |
| **Aggregation query** | Maps (untyped) | Text alias | `item.count` or `item.categoryName` |

```sail
/* Regular query - use field references */
local!cases: a!queryRecordType(
  fields: {'recordType!Case.fields.title'}
).data,
local!title: local!cases[1]['recordType!Case.fields.title']

/* Aggregation query - use text aliases */
local!counts: a!queryRecordType(
  fields: a!aggregationFields(
    measures: {a!measure(function: "COUNT", field: '...', alias: "total")}
  )
).data,
local!count: local!counts[1].total  /* or local!counts[1]["total"] */
```

---

## Common Patterns

### Dropdown Choices from Query

**Pattern:** Query lookup tables with `id` and `label` fields for component choices.

```sail
local!statusOptions: a!queryRecordType(
  recordType: 'recordType!Status',
  fields: {
    'recordType!Status.fields.id',      /* For choiceValues */
    'recordType!Status.fields.label'    /* For choiceLabels */
  },
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),  /* 100 for dropdown data */
  fetchTotalCount: true
).data,

a!dropdownField(
  label: "Status",
  choiceLabels: index(local!statusOptions, 'recordType!Status.fields.label', {}),
  choiceValues: index(local!statusOptions, 'recordType!Status.fields.id', {}),
  value: local!selectedStatusId,
  saveInto: local!selectedStatusId
)
```

**Why id + label:**
- `id` is used as the saved value (choiceValues)
- `label` is shown to the user (choiceLabels)
- Standard pattern for picker components, checkboxes, radio buttons, etc.

### Search Filter Pattern

```sail
local!searchText: null,

local!filteredCases: a!queryRecordType(
  recordType: 'recordType!Case',
  fields: { /* fields */ },
  filters: a!queryFilter(
    field: 'recordType!Case.fields.title',
    operator: "includes",
    value: local!searchText,
    applyWhen: a!isNotNullOrEmpty(local!searchText)
  ),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 100),
  fetchTotalCount: true
).data
```

### Date Range Filter

```sail
filters: a!queryLogicalExpression(
  operator: "AND",
  filters: {
    a!queryFilter(
      field: 'recordType!Case.fields.createdAt',
      operator: ">=",
      value: local!startDate,
      applyWhen: a!isNotNullOrEmpty(local!startDate)
    ),
    a!queryFilter(
      field: 'recordType!Case.fields.createdAt',
      operator: "<=",
      value: local!endDate,
      applyWhen: a!isNotNullOrEmpty(local!endDate)
    )
  }
)
```

---

## Anti-Patterns

### ❌ Missing fields Parameter

```sail
/* WRONG - Only returns primary key, all other fields will be null */
local!cases: a!queryRecordType(
  recordType: 'recordType!Case',
  filters: a!queryFilter(...),
  pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 10)
).data
```

### ❌ Using a!relatedRecordData() on Many-to-One Relationship

```sail
/* WRONG - status is many-to-one, not one-to-many */
a!queryRecordType(
  recordType: 'recordType!Case',
  relatedRecordData: {
    a!relatedRecordData(
      relationship: 'recordType!Case.relationships.status'  /* M:1 relationship! */
    )
  }
)
/* ERROR: "Only one-to-many type record type relationship references can be used" */

/* CORRECT - For M:1, use fields parameter */
a!queryRecordType(
  recordType: 'recordType!Case',
  fields: {
    'recordType!Case.relationships.status.fields.label'  /* Access M:1 via fields */
  }
)
```

### ❌ Missing limit on One-to-Many (Defaults to 10)

```sail
/* WRONG - Only returns first 10 notes per case */
fields: {
  'recordType!Case.relationships.notes.fields.description'
}
/* Returns only 10 notes by default! */

/* CORRECT - Specify limit with a!relatedRecordData() */
fields: {
  'recordType!Case.relationships.notes.fields.description'
},
relatedRecordData: {
  a!relatedRecordData(
    relationship: 'recordType!Case.relationships.notes',
    limit: 50  /* Explicit limit */
  )
}
```

### ❌ Wrong Field References in a!relatedRecordData() Filters

```sail
/* WRONG - Using base record type fields in related record filters */
a!relatedRecordData(
  relationship: 'recordType!Case.relationships.notes',
  filters: a!queryFilter(
    field: 'recordType!Case.fields.status',  /* ERROR - Case field, not Note field */
    operator: "=",
    value: "Open"
  )
)

/* CORRECT - Field references must be from the related record type */
a!relatedRecordData(
  relationship: 'recordType!Case.relationships.notes',
  filters: a!queryFilter(
    field: 'recordType!Note.fields.noteType',  /* Note field ✓ */
    operator: "=",
    value: "Internal"
  )
)
```

### ❌ Using "sorts" Instead of "sort"

```sail
/* WRONG - Parameter doesn't exist */
a!queryRecordType(
  sorts: {a!sortInfo(...)}  /* Invalid! */
)

/* CORRECT - sort goes inside pagingInfo */
a!queryRecordType(
  pagingInfo: a!pagingInfo(
    startIndex: 1,
    batchSize: 100,
    sort: {a!sortInfo(...)}
  )
)
```

### ❌ Nesting Logical Expressions Incorrectly

```sail
/* WRONG - Can't nest a!queryLogicalExpression in filters array */
filters: a!queryLogicalExpression(
  operator: "AND",
  filters: {
    a!queryFilter(...),
    a!queryLogicalExpression(...)  /* ERROR! */
  }
)

/* CORRECT - Use logicalExpressions parameter */
filters: a!queryLogicalExpression(
  operator: "AND",
  filters: {a!queryFilter(...)},
  logicalExpressions: {
    a!queryLogicalExpression(...)
  }
)
```

### ❌ Variable Filters Without applyWhen

```sail
/* WRONG - Filter will fail if local!status is null */
a!queryFilter(
  field: 'recordType!Case.fields.status',
  operator: "=",
  value: local!status
)

/* CORRECT - Conditionally apply when value exists */
a!queryFilter(
  field: 'recordType!Case.fields.status',
  operator: "=",
  value: local!status,
  applyWhen: a!isNotNullOrEmpty(local!status)
)
```

### ❌ Wrong Property Access for Aggregations

```sail
/* WRONG - Trying to use field references on aggregation result */
local!counts: a!queryRecordType(
  fields: a!aggregationFields(
    measures: {a!measure(function: "COUNT", field: '...', alias: "total")}
  )
).data,
local!count: local!counts[1]['recordType!...']  /* FAILS - this returns maps, not records */

/* CORRECT - Use text alias */
local!count: local!counts[1].total
```

---

## Quick Reference Checklists

### Choosing the Query Function

- [ ] Have a specific record identifier/PK? → Use `a!queryRecordByIdentifier()`
- [ ] Need aggregations (COUNT, SUM, AVG)? → Use `a!queryRecordType()` with `a!aggregationFields()`
- [ ] Need multiple records with filters? → Use `a!queryRecordType()`
- [ ] Need >100 related records? → Use `a!queryRecordByIdentifier()` (max 250) or separate query
- [ ] For grid/chart component? → Use `a!recordData()` in component's `data` parameter only

### When writing `a!queryRecordType()`

- [ ] Include `recordType` parameter
- [ ] Include `fields` parameter with ALL fields needed
- [ ] Include `pagingInfo: a!pagingInfo(startIndex: 1, batchSize: N)`
- [ ] Include `fetchTotalCount: true`
- [ ] Use `sort` (not `sorts`) **inside** pagingInfo
- [ ] Add `applyWhen` to filters with variable values
- [ ] Use `logicalExpressions` parameter for nested logic
- [ ] Check for empty results before accessing `.data`
- [ ] Use field references for regular queries, aliases for aggregations
- [ ] For **many-to-one** relationships: include in `fields` parameter
- [ ] For **one-to-many** relationships: use `a!relatedRecordData()` with explicit `limit`
- [ ] In `a!relatedRecordData()`: field references start from the related record type, not base
- [ ] Verify relationship type (1:M only) before using `a!relatedRecordData()`

### When writing `a!queryRecordByIdentifier()`

- [ ] Include `recordType` parameter
- [ ] Include `identifier` parameter (PK value or composite PK map)
- [ ] Include `fields` parameter (or only PK is returned)
- [ ] For composite PK: use `'recordType!Type'(field1: val1, field2: val2)` format
- [ ] Returns single record directly (NOT array) — no `[1]` needed
- [ ] For **one-to-many** relationships: use `a!relatedRecordData()` (max 250 vs 100)
- [ ] Do NOT use in loops — use `a!queryRecordType()` with `operator: "in"` instead
- [ ] Do NOT use for aggregations — use `a!queryRecordType()` with `a!aggregationFields()`
