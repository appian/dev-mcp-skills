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

## Configuration

**Appian Version:** The skill uses your Appian version for documentation lookups. Configure it in `skills/appian/SKILL.md`:

```markdown
## Configuration

**Appian Version:** 26.5
```

Change this value to match your Appian environment (26.5, 26.3, 25.4, 25.3, 25.2, 25.1, 24.4, or 24.3).

This affects:
- Documentation URL lookups when the skill searches Appian docs
- Function availability checks
- Version-specific guidance

### Documentation Search Examples

The skill automatically searches Appian documentation in three scenarios:

#### Scenario 1: Function Discovery (User describes what, not function name)
```
You: "Create an expression rule that calculates the distance in miles between 
     two locations. The rule accepts start latitude, start longitude, end 
     latitude, and end longitude."

AI: [Loads expressions.md, sees "Before Implementing Custom Logic" section]
    [Recognizes "distance" and "coordinates" as trigger keywords]
    [Searches functions.json: jq 'keys[] | select(test("distance"; "i"))']
    [Finds: a!distancebetween]
    [Fetches documentation from Appian docs]
    [Creates expression using built-in function + meter-to-mile conversion]

Result: Clean 20-line implementation using a!distanceBetween() instead of 
        40-line custom Haversine formula
```

**Common triggers:** distance/coordinates, encrypt/hash, working days, JSON/XML parsing, mathematical operations

#### Scenario 2: Explicit Function Lookup (User names the function)
```
You: "What parameters does a!startProcess() accept?"

AI: [Checks function-reference.md - not found]
    [Searches functions.json for "a!startprocess"]
    [Finds: /suite/help/26.5/Start_Process_Smart_Service.html]
    [Fetches documentation page]
    [Extracts signature and parameters]

Result: Complete function signature:
        a!startProcess(processModel, processParameters, isSynchronous, 
                       onSuccess, onError, onIncomplete)
        with descriptions for all 6 parameters
```

#### Scenario 3: Validation Error Recovery
```
You: [AI creates expression with wrong a!relatedRecordData() syntax]

AI: [validateExpression returns error: "Invalid parameter 'sortBy'"]
    [Automatically searches functions.json for a!relatedRecordData]
    [Fetches official documentation]
    [Corrects parameter: sortBy → sort]
    [Re-validates successfully]

Result: Fixed expression without manual doc lookup
```

**How it works:**
1. **58 curated functions** in reference files (common use cases, documented with patterns)
2. **437 additional functions** via automatic search (on-demand, as needed)
3. **Keyword-based discovery** before implementing custom logic (new!)
4. **Session caching** for fast subsequent lookups
5. **Tool-agnostic** bash/curl/jq commands (works across all AI tools)

See `skills/appian/SKILL.md` for the complete search workflow.

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

**Last version update:** 2026-06-25

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose improvements.

Skills are domain knowledge, not tool documentation. Good contributions:
- Name gotchas the tool schemas don't surface
- Show concrete patterns (correct and incorrect)
- Stay tool-agnostic — describe what to do, not which tool to call

## License

[Apache License 2.0](LICENSE)
