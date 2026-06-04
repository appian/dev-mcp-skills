# Skills

A skill is a markdown file that Claude Code loads on demand when its description matches the current task. Skills here teach Claude how to use the Appian MCP tools well — naming conventions, common patterns, gotchas, the difference between a record type field and a relationship, etc.

## File layout

```
skills/
  appian-<topic>/
    SKILL.md
```

Each skill lives in its own directory under `skills/`, and the directory name must start with `appian-` (the setup script symlinks all `skills/appian-*/` into `~/.claude/skills/`). The skill's content goes in `SKILL.md`.

## Required frontmatter

Every `SKILL.md` starts with YAML frontmatter:

```yaml
---
name: "appian-record-types"
description: "Create and modify Appian record types including fields, relationships, and source configuration. Covers field types, naming conventions, relationship management, and database table creation. Use when working with record types, data models, database tables, or record type relationships."
---
```

- **`name`** — must match the directory name exactly. This is how Claude Code references the skill.
- **`description`** — a few sentences. Claude reads this to decide whether the skill is relevant to the current task. It should answer "what does this skill cover" and "when should you use it." Be concrete: list the specific concepts, tools, or scenarios. Vague descriptions ("helps with Appian") get ignored because they match everything and nothing.

## What makes a good skill

1. **Tied to actual MCP tools.** Every skill in this repo opens with a "Relevant Tools" or similar section listing the tool names Claude will call. This grounds the prose in real capabilities — without it Claude guesses.
2. **Concrete patterns over principles.** "Use UPPER_SNAKE_CASE for table names" beats "follow naming conventions." Show one or two short examples of correct and incorrect usage.
3. **Names the gotchas.** Things that aren't obvious from the tool schemas — required ordering, hidden defaults, common mistakes. This is the highest-value content.
4. **Stays in one lane.** One skill = one topic. The 13 existing skills are roughly aligned with object types (record types, interfaces, process models, sites, etc.) and a few cross-cutting concerns (security, change planning, change review). Don't write a single mega-skill.
5. **Stays under ~300 lines.** Look at existing skills as a sizing reference. Long skills get partially loaded into context and the back half goes unread.

## Adding or editing a skill

1. Create or edit `skills/appian-<topic>/SKILL.md`.
2. Re-run `./setup.sh` from the repo root if you added a new skill — it symlinks the new directory into `~/.claude/skills/`. Editing an existing skill takes effect immediately (the symlink already points at the file).
3. Open a merge request. There's no automated quality gate; review is by reading.

## Testing a skill locally before submitting

Skills are loaded by Claude Code on demand based on their `description` matching the current task. To verify your new or edited skill loads and helps:

1. From the repo root, re-run `./setup.sh` (only required for *new* skills — re-creates the symlink).
2. Fully quit any running Claude Code sessions and relaunch via `./claude-appian`. Skills are read at startup; existing sessions won't see new ones.
3. In Claude Code, ask a prompt that should trigger your skill (e.g., for `appian-record-types`: "create a Customer record type with fields ID, name, email"). Watch for Claude referencing your skill's content — it'll often paraphrase or quote it.
4. If Claude doesn't seem to have loaded your skill, check that the `description` field is concrete enough to match the prompt. Vague descriptions get ignored.

## When *not* to write a skill

If the information is already in the MCP tool's schema (parameter docs, return shape), don't duplicate it in a skill. Skills are for the surrounding context the schema can't carry — *why*, *when*, and *which combinations*.
