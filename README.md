# linear-build

A Claude Code plugin that turns **Linear into your spec source** for building software, and bundles the **official Linear MCP server** so installing it wires up both the workflow and the integration.

> A Linear issue is the brief Claude reads *before* writing code — intent, acceptance criteria, and a verification checklist. Claude builds to it, proves it against real data, and writes the result back to the issue. A project = a milestone fanned into spec'd issues.

## What's in the box

- **Skill** (`skills/linear-build/`) — the workflow: build an issue, create an issue from intent, or kick off a whole project.
- **Linear MCP server** (`.mcp.json`) — the official `https://mcp.linear.app/mcp` (Streamable HTTP, OAuth 2.1), registered as `linear-server`.
- **Guide** (`docs/linear-with-claude-code.md`) — the plain-English deep reference.

## The three modes

| Trigger | What happens |
|---|---|
| `build RAD-42` / any `TEAM-123` | Read the issue → align → build to its acceptance criteria → verification gate → close with evidence |
| `turn this into a Linear issue` | Draft the Intent / Acceptance / Verification spec, confirm, create it |
| `create a Linear project for X` | Create the project, fan it into spec'd issues, confirm the plan, then build the first |

## Install

This repo is a self-contained Claude Code plugin marketplace.

```bash
# 1. Add this repo as a marketplace (local path or git URL)
/plugin marketplace add ~/Documents/Projects/linear-build-plugin
#    or, once pushed to GitHub:
#    /plugin marketplace add <your-github-user>/linear-build-plugin

# 2. Install the plugin
/plugin install linear-build@tarik-skills

# 3. Restart Claude Code so the skill + MCP server load
```

### First-time Linear auth

The Linear MCP uses OAuth. After install + restart, run:

```
/mcp
```

and complete the Linear sign-in for the `linear-server` connection. (Tools appear under the `mcp__linear-server__*` namespace.)

### Manual install (no plugin system)

If you'd rather not use the plugin system:

```bash
# copy the skill
cp -r skills/linear-build ~/.claude/skills/

# add the MCP server (official setup command from Linear's docs)
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
# then run /mcp in a session to authenticate
```

## The issue-as-spec contract

Every issue should carry these sections — that's what makes the workflow worth it:

```markdown
## Intent
What you want and why. One or two plain sentences.

## Acceptance criteria
The finish line — checkable bullets.
- [ ] ...

## Verification checklist (prove it against reality)
How we'll prove each criterion with real data, not assertions.
- [ ] ...

## Out of scope
What this issue is NOT.
```

## Why

Specs that live in chat disappear, have no agreed finish line, and are ad hoc. A Linear issue fixes all three — and its verification checklist doubles as the proof gate before anything ships. See [`docs/linear-with-claude-code.md`](docs/linear-with-claude-code.md).

---

Official Linear MCP docs: https://linear.app/docs/mcp
