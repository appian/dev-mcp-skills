# Composer Record Types Guidance

**Source:** `/Users/jupiter.munoz/repo/composer/service-components/developer-copilot/src/developer_copilot/orchestrators/aip/`

**Status:** TEMPORARY — extracted during Phase 1 integration

**Purpose:** This folder contains raw Composer guidance files for comparison and validation during Week 1 integration. These files represent battle-tested patterns from the Composer backend.

**Timeline:** This folder will be deleted at Phase 1.5 start once integration is validated and patterns are fully extracted into our skill files.

## File Contents

- `planning.md` — 238 lines of record type planning guidance (Appian-specific platform rules, naming conventions, field ordering)
- `execution.md` — 591 lines of record type execution guidance (MCP tool mechanics, sample data generation, record title expressions)

## Usage

These files are reference material during the rewrite of `appian-record-types/SKILL.md`. The skill will extract key patterns from both planning and execution layers.

## Key Patterns to Extract

### From planning.md:
- Record type naming conventions (WITH application prefix)
- Table naming conventions (WITH application prefix)
- Field ordering for UI layout
- Record descriptions and field descriptions

### From execution.md:
- Sample data quantity guidance (15-20 entities, 20-30 junction relationships)
- Sample data distribution patterns
- Record title expression setup
- Bidirectional relationship mechanics
- Reference table mandatory structure (4 fields)

## Related

- Main skill: `../../SKILL.md`
- Data modeling guidance: `../../../appian-data-modeling/references/composer/`
