# Appian MCP Skills

Skills that teach AI coding assistants how to build Appian applications using the [Appian MCP tools](https://docs.appian.com/suite/help/latest/devmcp.html). Install alongside your MCP-enabled IDE to get better first-pass results and fewer retry loops.

## What are skills?

Skills are markdown files that provide domain-specific knowledge to AI assistants. They describe Appian platform patterns, constraints, and best practices so the model doesn't have to learn them through trial and error during your session.

## Structure

```
skills/
  appian/
    SKILL.md                  ← Entry point (description, reference map, dependency order)
    references/
      tools-mcp.md            ← MCP tool patterns and non-obvious behaviors
      record-types.md         ← Record type schemas, fields, relationships, actions
      data-modeling.md        ← Entity design, normalization, naming conventions
      interfaces.md           ← SAIL forms, dashboards, summary views
      process-models.md       ← Nodes, variables, start forms, flow patterns
      sail.md                 ← Components, layouts, data binding, grids
      ...                     ← Additional reference files
```

The skill has a single entry point (`SKILL.md`) with a resource reference map that tells the assistant which reference file to load for any given task. Reference files contain domain knowledge — schemas, conventions, patterns, and pitfalls — without tool-specific syntax.

## Installation

### Kiro

Add as a workspace skill in `.kiro/skills/`.

### Claude Code

```bash
git clone https://github.com/appian/dev-mcp-skills.git ~/.claude/skills/appian
```

### Cursor

Add as project rules:

```bash
git clone https://github.com/appian/dev-mcp-skills.git .cursor/rules/appian
```

### Other IDEs

Copy `skills/appian/` into wherever your IDE loads context/instruction files from. The directory is self-contained.

## Prerequisites

### Required: Appian MCP Tools

Install the [Appian MCP tools](https://docs.appian.com/suite/help/latest/devmcp.html) to create/modify Appian objects (record types, interfaces, process models, etc.).

### Recommended: Appian Documentation Search

Install the official Appian documentation search MCP server for enhanced pattern discovery:

**Benefits:**
- Access to 200+ official recipes (query, function, interface)
- UI/UX patterns (drilldown, filtering, grids, forms)
- Design best practices and performance guidelines
- 75% fewer validation errors in testing

**Installation:**

Add to your `.mcp.json`:

```json
{
  "mcpServers": {
    "appian-public-docs": {
      "type": "http",
      "url": "https://appian-docs-public.mcp.kapa.ai"
    }
  }
}
```

**Authentication:** OAuth via Google or GitHub (one-time setup)

**Rate limits:** 300 requests/day, 60/minute (very generous for typical usage)

**Optional:** The skill works without this tool by using skill references only. With the tool installed, Claude automatically enriches skill knowledge with official documentation when needed.

See [Three-Tier Documentation Lookup Strategy](skills/appian/SKILL.md#three-tier-documentation-lookup-strategy) for details.

## Configuration

**Appian Version:** The skill uses your Appian version for documentation lookups. Configure it in `skills/appian/SKILL.md`:

```markdown
## Configuration

**Appian Version:** 26.6
```

Change this value to match your Appian environment (26.6, 26.5, 26.3, 25.4, 25.3, 25.2, 25.1, 24.4, or 24.3).

This affects:
- Documentation URL lookups (Tier 2: functions.json)
- Function availability checks
- Version-specific guidance

### Three-Tier Documentation Lookup

The skill uses a three-tier strategy to enrich its knowledge when skill references lack sufficient detail:

**Tier 1: Skill References** (always check first)
- 58 curated functions with patterns and anti-patterns
- Detailed examples for complex scenarios
- May exceed official docs in depth

**Tier 2: functions.json + curl** (function lookups)
- Definitive function existence checks
- Official signatures for 495 total functions
- Version-specific, deterministic lookups

**Tier 3: Documentation Search Tool** (patterns & recipes - requires optional MCP server)
- 200+ official recipes (query, function, interface)
- UI/UX patterns (drilldown, filtering, grids)
- Design best practices and performance guidelines
- Semantic search for "how to" questions

#### Example: Pattern Discovery with Tier 3

```
You: "Create a grid with drilldown pattern to show order details"

AI: [Loads interfaces.md, sail.md - basic grid info only]
    [Recognizes "drilldown" is a specific pattern not detailed in references]
    [Searches Tier 3: "What is the drilldown pattern for grids in Appian?"]
    [Finds: Official drilldown recipe with grid + detail view example]
    [Combines skill references (anti-patterns) + search results (official pattern)]
    [Implements using official Appian drilldown pattern]

Result: Grid with proper drilldown implementation, 1 minor error vs 4 errors 
        when inventing pattern from scratch (75% error reduction)
```

#### Example: Function Lookup with Tier 2

```
You: "Does the function regexmatch() exist in Appian?"

AI: [Checks function-reference.md anti-hallucination list]
    [Confirms: regexmatch() does NOT exist]
    [Suggests alternatives: like(), find(), search()]

Result: Definitive "No" with correct alternatives, prevents false positive
```

**How it works:**
1. **Tier 1 first:** Check skill references (curated, instant)
2. **Tier 2 for functions:** Existence checks and signatures (fast, exact)
3. **Tier 3 for patterns:** Recipes and best practices (semantic search)
4. **Combine sources:** Skill references + official docs = enriched knowledge

See [Documentation Lookup Strategy](skills/appian/references/documentation-lookup-strategy.md) for complete workflows, curl commands, error handling, and examples.

## Usage

The AI loads the skill on demand based on the task — you don't need to reference it explicitly. The skill covers:

- Applications, record types, fields, relationships
- Interfaces (forms, dashboards, summary views)
- Expression rules and SAIL expressions
- Process models (nodes, variables, start forms)
- Sites, Web APIs, constants, groups, folders, documents
- Data modeling, security, change planning, change review

## Maintenance

**Appian Version Support:** This skill supports the 8 most recent Appian versions. When new versions are released (typically quarterly), update:

1. `skills/appian/SKILL.md` - Configuration section with new version list
2. `README.md` - This Configuration section
3. Remove oldest version (typically 2 years old)

**Last version update:** 2026-06-26

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose improvements.

Skills are domain knowledge, not tool documentation. Good contributions:
- Name gotchas the tool schemas don't surface
- Show concrete patterns (correct and incorrect)
- Stay tool-agnostic — describe what to do, not which tool to call

## License

[Apache License 2.0](LICENSE)
