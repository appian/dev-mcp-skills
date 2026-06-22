# AGENTS.md

This repository contains a skill that teaches AI coding agents how to build Appian applications. The skill content is domain knowledge — platform conventions, patterns, and gotchas — that the agent can't infer from tool schemas alone.

## Git Workflow

- Work on feature branches off `main`. Never commit directly to `main`.
- Push to your fork only. Use pull requests to propose changes to the upstream repository.
- Do not force push to any shared branch.
- Do not amend commits that have already been pushed.

## Content Principles

### Sizing

- `SKILL.md` should stay under 500 lines — it's always loaded when the skill triggers
- Reference files can be longer but for files over 300 lines, include a table of contents at the top
- If a reference is growing past useful size, split into focused files and add rows to the resource reference map

### Writing style

- Use imperative form for instructions
- Explain why things matter rather than piling on MUSTs and NEVERs
- Be concrete — show correct and incorrect examples
- Stay general enough to apply across projects, not narrow to one specific scenario

### Tool-agnostic references

All files under `skills/appian/references/` (except `tools-mcp.md`) must be tool-agnostic:
- Describe what to do, not which tool to call
- No MCP tool names (e.g., `createRecordType`, `addRecordTypeRelationship`)
- No CLI commands or flags
- Use generic language: "create the record type", "list fields", "get the configuration"

Only `references/tools-mcp.md` may reference MCP tool names. It covers patterns and non-obvious behaviors — not a catalog of available tools (the MCP schema already provides that).

### Don't duplicate tool metadata

The MCP tool schemas describe parameters, types, and return values. Reference files should cover what schemas cannot: conventions, patterns, gotchas, ordering constraints, and domain knowledge.

### Skill content vs. tool issues

The skill teaches durable domain knowledge — patterns and conventions that are true regardless of tool implementation. Do not encode workarounds for tool bugs or limitations in the skill. If a failure is caused by the tool itself (e.g., validation behavior that doesn't match Appian Designer, missing support for an object type), file a GitHub issue with the `tool-change-proposal` label instead. See CONTRIBUTING.md for the issue format.

## Skill Structure

```
skills/appian/
  SKILL.md                  ← Entry point (reference map, dependency order)
  references/
    tools-mcp.md            ← MCP tool patterns (only file with tool names)
    record-types.md         ← Domain reference
    data-modeling.md        ← Domain reference
    ...
```

### What goes in SKILL.md

SKILL.md is the routing layer. It tells the agent what's available and where to find it. It contains:

- Frontmatter (name, description)
- Tool surface discovery (how to identify available tools)
- Resource reference map (which file to load for which task)
- Loading strategy (what order to load references)
- Dependency order (what order to create objects)
- A short tips section (5-10 universal rules that apply regardless of task)

SKILL.md does NOT contain domain knowledge, schemas, examples, patterns, or troubleshooting. Those go in reference files.

### What goes in reference files

Reference files contain the actual domain knowledge for a specific topic. Each file should be self-contained for its area:

- JSON schemas and field definitions
- Naming conventions specific to that object type
- Design patterns and structural guidance
- Common pitfalls and how to avoid them
- Examples showing correct usage

If you're adding content, find the right reference file and add it there. If no file fits, propose a new one — but it must also get a row in the resource reference map.

### Adding content

- Adding a new reference file requires a corresponding row in the resource reference map in `SKILL.md`
- Never add domain content to SKILL.md directly — find or create the right reference file
- If a reference file is growing unwieldy, split it rather than trimming useful content

## Pull Request Expectations

- One concern per PR
- Commit messages should be clear and intentional
- Follow the PR template (`.github/pull_request_template.md`)
- Do not include dev tooling, test harnesses, or internal artifacts
- Before pushing, check whether your changes make any existing content outdated (README.md, AGENTS.md, CONTRIBUTING.md, SKILL.md resource reference map). Update them in the same PR.
