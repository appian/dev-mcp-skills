# Appian MCP Skills

Skills that teach AI coding assistants how to build Appian applications using the [Appian MCP tools](https://docs.appian.com/suite/help/latest/devmcp.html). Install these alongside your MCP-enabled IDE (Claude Code, Cursor, Windsurf, etc.) to get better first-pass results and fewer retry loops.

## What are skills?

Skills are markdown files that provide domain-specific knowledge to AI assistants. They describe Appian platform patterns, constraints, and best practices so the model doesn't have to learn them through trial and error during your session.

## Installation

### Claude Code

```bash
# Clone into your CLAUDE.md skills directory
git clone https://github.com/appian/dev-mcp-skills.git ~/.claude/skills/appian

# Or symlink individual skills
ln -s /path/to/dev-mcp-skills/skills/appian-sail ~/.claude/skills/appian-sail
```

### Cursor

Add the skills as project rules. In your project's `.cursor/rules/` directory:

```bash
git clone https://github.com/appian/dev-mcp-skills.git .cursor/rules/appian
```

Or symlink individual skills into `.cursor/rules/`.

### VS Code (Copilot)

Add the skills as workspace instructions. Clone into `.github/copilot-instructions/`:

```bash
git clone https://github.com/appian/dev-mcp-skills.git .github/copilot-instructions/appian
```

### Other IDEs

Copy the `skills/` directory contents into wherever your IDE loads context/instruction files from. Each `appian-*/` folder is self-contained.

## Skills Included

| Skill | Purpose |
|-------|---------|
| `appian-change-planning` | Discover what exists, determine scope, order dependencies |
| `appian-change-review` | Verify changes landed correctly, detect discrepancies, runtime checks |
| `appian-data-modeling` | Entity design, normalization, relationships, naming conventions |
| `appian-expression-rules` | Rule inputs, expression bodies, when to use rules vs. inline |
| `appian-expressions` | Syntax, operators, type system, object references, common functions |
| `appian-interfaces` | Interface lifecycle, dashboard/form design, inputs |
| `appian-process-models` | Nodes, variables, connections, start forms, flow patterns |
| `appian-record-types` | Fields, relationships, source configuration, database tables |
| `appian-sail` | Components, layouts, data binding, grids, links, platform quirks |
| `appian-security` | Group hierarchies, role-based access, object permissions |
| `appian-sites` | Site creation, pages, navigation, visibility |
| `appian-supporting-objects` | Constants, groups, folders, documents |
| `appian-web-apis` | HTTP endpoints, URL aliases, expression bodies |

## Skill Structure

Each skill is a directory containing:

```
appian-<topic>/
  SKILL.md              ← Main content (YAML frontmatter + markdown)
  references/           ← Optional deep-reference material (loaded on demand)
    *.md
```

The `SKILL.md` frontmatter declares the skill name and a description that tells the AI when to load it:

```yaml
---
name: "appian-sail"
description: "Write SAIL code for Appian interfaces — components, layouts, ..."
---
```

## Usage Tips

- Install all 13 skills for general Appian development
- The AI loads skills on demand based on the task — you don't need to reference them explicitly
- Skills are additive — they improve outcomes but don't change what tools are available
- If the AI still gets something wrong, that's a candidate for a new skill contribution

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose new skills or improve existing ones.

## License

[Apache License 2.0](LICENSE)
