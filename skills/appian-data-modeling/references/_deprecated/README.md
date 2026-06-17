# Deprecated Data Modeling References

**Status:** DEPRECATED — replaced by Composer-based guidance

**Reason for deprecation:**

These reference files contained incorrect patterns discovered during Week 1 Composer integration:

1. **entity-patterns.md.old** — Primary key naming error (`case_id` instead of `id`)
2. **relationship-patterns.md.old** — Primary key naming error, incomplete reference table structure

## What Was Wrong

### Primary Key Naming
- **Incorrect:** `[table_name_lowercase]_id` (e.g., `case_id`, `customer_id`)
- **Correct:** `id` (Appian platform requirement)

### Reference Table Structure
- **Incomplete:** id, code, name (3 fields)
- **Correct:** id, label, sortOrder, isActive (4 mandatory fields)

### Table Naming
- **Missing:** Application prefix requirement
- **Correct:** Table names should include application prefix (e.g., `JTA_CASE` not `CASE`)

## Replaced By

These files were replaced by Composer guidance extracted from `/Users/jupiter.munoz/repo/composer`:
- Planning layer: `planning/prompts/data_model.md`
- Execution layer: `execution/prompts/record_type.md`

See `../composer/README.md` for current reference material.

## Timeline

- Deprecated: 2026-06-17 (Week 1, Day 1)
- Kept for historical reference only — do not use these patterns
