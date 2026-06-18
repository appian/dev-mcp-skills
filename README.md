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

## Usage

The AI loads the skill on demand based on the task — you don't need to reference it explicitly. The skill covers:

- Applications, record types, fields, relationships
- Interfaces (forms, dashboards, summary views)
- Expression rules and SAIL expressions
- Process models (nodes, variables, start forms)
- Sites, Web APIs, constants, groups, folders, documents
- Data modeling, security, change planning, change review

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose improvements.

Skills are domain knowledge, not tool documentation. Good contributions:
- Name gotchas the tool schemas don't surface
- Show concrete patterns (correct and incorrect)
- Stay tool-agnostic — describe what to do, not which tool to call

## License

[Apache License 2.0](LICENSE)
