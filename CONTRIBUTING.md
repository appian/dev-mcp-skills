# Contributing to Appian MCP Skills

We welcome contributions from anyone building with Appian and AI coding assistants. Whether you're an Appian employee, a partner, or a practitioner — if you've found a pattern that makes the AI more reliable, we want it here.

## How to Contribute

1. **Fork** this repository
2. **Create a branch** in your fork (`git checkout -b my-improvement`)
3. **Make your changes** (see guidelines below)
4. **Commit with DCO sign-off** (see below)
5. **Open a Pull Request** against `main`

## What Makes a Good Contribution

### Evidence of impact

Every change should address a real failure mode. In your PR description, include:

- **What went wrong** — describe the error, retry loop, or incorrect output the AI produced
- **Why** — what knowledge was the AI missing?
- **How this fixes it** — what does the skill now teach that prevents the failure?

You don't need to provide formal metrics, but "I hit this problem, added this content, and the AI stopped making the mistake" is the bar.

### Scope

- **One concern per PR** — don't bundle unrelated fixes
- **Add to existing reference files** — most contributions improve an existing reference (e.g., adding a pitfall to `references/record-types.md`)
- **New reference files** are rare — only when a topic is substantial enough to warrant its own file and doesn't fit in any existing reference

### Quality

- Write for an AI audience — be precise and explicit, not conversational
- Stay tool-agnostic — describe what to do, not which tool to call (no MCP tool names or CLI commands in reference content)
- Include concrete examples where applicable
- State what NOT to do when there's a common wrong approach
- Use the existing reference files as style examples

## File Structure

```
skills/appian/
  SKILL.md              ← Entry point (description, reference map, dependency order)
  references/
    tools-mcp.md        ← MCP tool patterns and non-obvious behaviors
    record-types.md     ← Record type schemas, fields, relationships
    data-modeling.md    ← Entity design, normalization, naming conventions
    ...                 ← Additional reference files
```

`SKILL.md` has the resource reference map that tells the AI which file to load for a given task. If you add a new reference file, add a corresponding row to the map.

## DCO Sign-Off

This project uses the [Developer Certificate of Origin](https://developercertificate.org/) (DCO). You must sign off every commit to certify you have the right to submit it:

```bash
git commit -s -m "Add grid filtering quirk to references/sail.md"
```

This adds a `Signed-off-by: Your Name <your@email.com>` line. If you forget, amend:

```bash
git commit --amend -s --no-edit
```

All commits in a PR must have sign-off or the PR cannot be merged.

## Review Process

1. A maintainer will review your PR, usually within a few business days
2. We may ask for evidence of the failure mode or suggest a different reference file
3. Once approved, we'll squash-merge to keep `main` history clean

## Reporting Issues

If a reference teaches something **incorrect** (causes failures rather than preventing them), open an issue with:

- Which reference file and which section
- What it says vs. what actually works
- Your Appian version (if relevant)

## Tool Change Proposals

Sometimes a failure mode is caused by the MCP tools themselves — not missing skill knowledge. If the agent hits an error that can only be fixed by changing tool behavior (not by teaching a different pattern), open an issue with the `tool-change-proposal` label instead of submitting a skill change.

**When to file a tool change proposal (not a skill change):**
- The tool rejects valid inputs or configurations
- The tool's validation doesn't match in-product (Appian Designer) behavior
- A supported object type or field type is missing from the tool
- The tool returns incorrect data or silently drops configuration

**Issue format:**
- **Problem** — what operation fails and what error is returned
- **Evidence** — sample expressions or configurations that reproduce the issue, with the error messages. Do not include environment-specific names, server URLs, or internal identifiers.
- **Expected behavior** — what should happen instead
- **Impact** — what the agent can't automate as a result

Tool change proposals are triaged by maintainers who have access to the tool implementation. The skill should NOT encode workarounds for tool bugs — use the general "Handling Tool Limitations" guidance in `references/tools-mcp.md` instead.

## Code of Conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md). Be respectful and constructive.
