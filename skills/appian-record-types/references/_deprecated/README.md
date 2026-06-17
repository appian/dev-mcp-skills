# Deprecated Record Types References

**Status:** DEPRECATED — replaced by Composer-based guidance

**Timeline:** This folder is reserved for deprecated reference files during Week 1 Composer integration. Currently empty as record types skill had no separate reference files to deprecate (all patterns were in SKILL.md).

## What Will Be Deprecated

When rewriting `appian-record-types/SKILL.md` in Days 2-3, any extracted sections that are superseded by Composer guidance will be documented here.

## Known Pattern Updates

The following patterns from the original skill are being updated with Composer guidance:

### Naming Conventions
- **Old:** Record type names without application prefix ("Case", "Status", "Priority")
- **New:** Record type names WITH application prefix ("JTA Case", "JTA Status", "JTA Priority")

### Reference Table Structure
- **Old:** Not explicitly specified in skill
- **New:** MANDATORY 4 fields (id, label, sortOrder, isActive)

### Sample Data
- **Old:** Generic "populate reference tables" guidance
- **New:** Specific quantity guidance (15-20 entities, 20-30 junction, distribution patterns)

### Record Title Expression
- **Old:** Not mentioned in skill
- **New:** Explicit setup step after record type creation

## Replaced By

Composer guidance from:
- Planning layer: `../composer/planning.md`
- Execution layer: `../composer/execution.md`

## Related

- Current reference: `../composer/README.md`
- Data modeling deprecated: `../../../appian-data-modeling/references/_deprecated/`
