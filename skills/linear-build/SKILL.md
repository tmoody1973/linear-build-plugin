---
name: linear-build
description: Use when starting or building tracked work in Linear — e.g. "build RAD-42", "start the next Linear issue", "turn this into a Linear issue", or "create a Linear project for X / plan this milestone in Linear". Creates a project, fans it into spec'd issues (Intent / Acceptance / Verification) assigned to that project, builds each to its acceptance criteria, runs a verification gate against real data, and writes results back to the issue. Requires the Linear MCP server (linear-server).
---

# Linear-driven build loop

This build process uses **Linear as the spec source**. A Linear issue is the brief you read *before* writing code: it carries the intent, the acceptance criteria (the agreed finish line), and a verification checklist (how we prove it against real data). This skill operationalizes the guide bundled at `docs/linear-with-claude-code.md`.

Core principle: **the issue is the contract.** Don't freelance past it, and don't claim "done" until its verification checklist passes against reality.

## When this fires

- "build RAD-42" / "start RAD-42" / any `TEAM-123` issue reference → **Build an existing issue** (below).
- "start the next issue" / "what's next" → call `list_my_issues`, pick the top unblocked one, confirm, then build it.
- "turn this into a Linear issue" / "make an issue for this" → **Create an issue from intent** (below).
- "create a Linear project for X" / "plan this milestone" / a multi-phase feature → **Start a project (kickoff)** (below).
- Beginning a multi-step, load-bearing build with no issue yet → offer to create a project or an issue first.

Do NOT use this for typos, copy tweaks, or quick exploration. Linear is for consequential, multi-step work where a written finish line pays off.

## Linear MCP tools (namespace: `mcp__linear-server__*`)

If these aren't callable, the Linear MCP server isn't loaded in this session — tell the user to restart Claude Code. Key tools:

- `get_issue` — read one issue (id or `TEAM-123` identifier).
- `list_my_issues` — the user's assigned issues (for "what's next").
- `list_issues` — filter by team/project/status/label.
- `create_issue` — new issue (title, description, team; set **`project`** to assign it to a project; optional labels/assignee).
- `update_issue` — change state/description/assignee, or set **`project`** to move an existing issue into a project (Todo→In Progress→Done).
- `create_project` / `update_project` / `get_project` — create a project (name, description, team(s), optional lead/target date) and amend it.
- `list_comments` / `create_comment` — read/post issue comments (verification evidence goes here).
- `list_teams`, `list_projects`, `list_issue_statuses`, `list_issue_labels` — for resolving names→ids when creating/moving.

Resolve human names to ids with the `list_*` tools; never guess an id.

## The issue-as-spec contract

Every issue should carry these three sections. If the issue you're handed is missing them, **stop and offer to fill them in first** (propose the acceptance + verification, get a yes, `update_issue`), because the rest of the loop depends on them.

```markdown
## Intent
What the user wants and why. One or two plain sentences.

## Acceptance criteria
The finish line — checkable bullets.
- [ ] ...

## Verification checklist (prove it against reality)
How we'll prove each criterion with real data, not assertions.
- [ ] ...

## Out of scope
What this issue is NOT.
```

## Build an existing issue

1. **Read it.** `get_issue` for the identifier. If it lacks the spec sections, offer to add them and `update_issue` before building.
2. **Align out loud.** Restate the Intent and Acceptance criteria back to the user in 2-4 lines, then start. This catches a wrong-issue or scope mismatch before any code.
3. **Move it.** `update_issue` → In Progress.
4. **Build to the criteria.** Implement only what the acceptance criteria require (respect Out of scope). Commit atomically, referencing the identifier so git ↔ Linear stay linked:
   `feat(scope): short description (RAD-42)`
5. **Verification gate — the learning loop.** Before claiming done, for any **load-bearing** change (schema/data-model, money/totals, anything where a bug is expensive): show the user the **diff or the data model** and ask *"looking at the acceptance criteria, what would you check first?"* Compare their answer to what you actually checked. Then run the verification checklist against **real data** (real API/db/site), and show the evidence (screenshots, diffs, real numbers) — never assert.
6. **Close with proof.** When the checklist passes, `update_issue` → Done and `create_comment` with a one-line evidence note ("card total = $11, matched Funraise dashboard, screenshot attached"). Now the issue is a permanent record of *what* shipped and *how we knew it worked*.

## Create an issue from intent

When the user describes work and wants it tracked:

1. Resolve the team (`list_teams`; if more than one, ask which).
2. Draft the issue body using the **issue-as-spec contract** — write sharp Acceptance criteria and a real Verification checklist from their intent. Don't leave those sections empty; that's the whole value.
3. Show the draft, get a yes (tweak if needed), then `create_issue`.
4. Hand back the new `TEAM-123` identifier. Offer to build it now (→ Build an existing issue).

## Start a project (kickoff)

When the user wants to set up a new build effort — "create a Linear project for X", "plan this milestone", a multi-phase feature:

1. **Resolve the team** (`list_teams`; if more than one, ask which). Scan `list_projects` to avoid a duplicate.
2. **Define the project.** Name it for the outcome. Write a short project description stating the goal and the overall done condition (the milestone's acceptance) — the project is the parent spec.
3. **Break it into a small set of issues** — one shippable thing each, in dependency order (like phases). Draft every issue with the issue-as-spec contract (Intent / Acceptance / Verification). Don't leave those sections empty; that's the value.
4. **Present the plan and STOP for a yes.** Show the project + the issue list (titles + one-line intents) before creating anything. Let the user cut, reorder, or sharpen.
5. **Create it.** `create_project`, then `create_issue` for each with `project` set to the new project id so every issue is assigned to it. Set order/dependencies where it matters.
6. **Hand back** the project + issue identifiers, and offer to build the first one (→ Build an existing issue).

A Linear project here is the equivalent of a roadmap/phase set: parent = the milestone, issues = the phases/tasks under it.

## Guardrails (standing rules)

- **Prove against real data.** Trust evidence, not "it's verified." Every verification step shows real output.
- **Load-bearing changes get the diff-question.** Surface the diff/data model and ask what the user would check before you assert it passes. Not for cosmetic changes.
- **Keep issues small.** One issue = one shippable thing. Split anything bigger.
- **Cost/effort-aware.** Flag method and cost concerns early; avoid burning real money/test artifacts to verify when real existing data will do.
- **Decision-shaped forks, not code walkthroughs.** When there's a real choice, give the tradeoff, not a line-by-line tour.

Deep reference: `docs/linear-with-claude-code.md`. Companion idea: branch + PR is where a Linear issue gets built and previewed before merge.
