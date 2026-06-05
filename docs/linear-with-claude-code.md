# Using Linear with Claude Code — A Plain-English Guide

> **For:** Tarik, on the `rm-microsites` project
> **Your choice:** Linear is your **spec source** — each issue is the brief Claude reads before building. Not just a to-do list, and not (yet) driving the whole loop on its own.
> **The one-line idea:** Instead of describing what you want in chat every time, you write it once in a Linear issue. Claude reads that issue, builds to it, and you check the result against the same issue.

---

## Why bother (especially for you)

Right now your "spec" lives in chat messages: *"track a giving form amount," "make the 50/50 split bigger."* That works, but it has three weaknesses:

1. **It disappears.** Once the chat is gone, the reasoning is gone.
2. **There's no agreed finish line.** "Done" is whatever I assert. You want proof against a standard.
3. **It's ad hoc.** You said you want to engineer your orchestration instead of winging it.

A Linear issue fixes all three. It's a written brief that says: *here's what I want, here's how we'll know it's done, here's what to check.* That last part — the checklist — is the same thing that powers your verification learning loop. So Linear isn't extra overhead; it's the place your "show me proof" instinct lives permanently.

Think of it like a work order at the station: nobody starts the job until the order says what "finished and correct" looks like.

---

## What Linear is (30-second version)

Linear is an issue tracker — a clean, fast app for writing down units of work ("issues") and moving them through states (Todo → In Progress → Done). Each issue has:
- A **title** (short name)
- A **description** (the brief — this is where your spec goes)
- A **state**, an **assignee**, **labels**, and a **unique ID** like `RAD-42`

That ID (`RAD-42`) is the magic handle. You'll use it to tell Claude "go build this," and it shows up in your git commits and PRs so everything links back.

---

## The setup (one time)

Linear connects to Claude Code through an **MCP server** — think of MCP as a standard plug that lets Claude read and write your Linear issues directly, instead of you copy-pasting.

1. **Create a Linear account + workspace** (free tier is fine). Make a team — e.g. "Radio Milwaukee" — which gives you an issue prefix like `RAD-`.
2. **Connect Linear's MCP server to Claude Code.** Linear publishes an official MCP server. In Claude Code you add it as a connector; you'll authenticate once in the browser (sign in to Linear, approve access). After that, Claude can list, read, create, and update your issues.
   - When it's connected, you'll see Linear tools available to me (names like `linear-create-issue`, `linear-update-issue`, `linear-get-issue`).
   - If you want, just say *"connect Linear"* in a session and I'll walk you through the exact connector steps for your setup and confirm it's working.
3. **That's it.** No code changes to `rm-microsites`. Linear lives alongside the project, not inside it.

---

## The issue-as-spec template (the important part)

Because Linear is your **spec source**, every issue should follow the same shape. Copy this into the description field. This template is the whole point — it's your context engineering, written down.

```markdown
## Intent
What I actually want and why. One or two plain sentences.
(e.g. "Show a live total for the Anti-Gala giving form on the HYFIN page,
so the host can read it out during the event.")

## Acceptance criteria
The finish line. Bullet points. Each one is checkable.
- [ ] Card shows the form's current total, updated live
- [ ] Total matches what Funraise's dashboard shows (within one refresh)
- [ ] Anonymous gifts are counted but never show a name
- [ ] Nothing breaks if the form has zero gifts yet

## Verification checklist (how we PROVE it, against reality)
The "show me proof" list. This is what I'll walk you through before saying "done."
- [ ] Compare card total to the real Funraise form total — screenshot both
- [ ] Send one real test gift, confirm the card moves
- [ ] Check the data model: what field holds the total, and can it double-count?

## Out of scope
What this issue is NOT. Stops scope creep.
- Not building refunds handling
- Not touching the Givebutter card
```

The three sections map exactly to how you already work:
- **Intent** = the WHAT/WHY you own.
- **Acceptance criteria** = the agreed finish line (so "done" isn't just my word).
- **Verification checklist** = your "are you confirming this against real data?" — but written down in advance, so I have to meet it.

---

## The everyday loop

Here's the actual workflow once it's set up. Five steps.

**1. You write the issue in Linear.** Use the template. Spend 2 minutes. You can even draft it rough and ask me to tighten the acceptance criteria — *"here's issue RAD-42, sharpen the verification checklist."*

**2. You point me at it.** In a Claude Code session, say:
> "Build RAD-42."

I'll read the issue through the Linear connector, restate the intent and acceptance criteria back to you (so we're aligned before any code), then start.

**3. I build, and I move the issue.** I set it to *In Progress*, do the work, and commit using the issue ID so git and Linear stay linked:
> `feat(funraise): live form total card (RAD-42)`

**4. The verification gate — your learning loop.** Before I claim it's done, for anything load-bearing I'll show you the **diff or the data model** and ask: *"Here's the change. Looking at the acceptance criteria, what would you check first?"* We compare your answer to what I actually checked. Then we run the verification checklist together against real data.

**5. We close it.** When the checklist passes, the issue moves to *Done*, with a short note of what proved it (e.g. "card total = $11, matched Funraise dashboard, screenshots attached"). Now there's a permanent record of *what_ was built and _how we knew it worked*.

---

## A worked example (this week's raffle card)

How the raffle card would have looked as a Linear issue:

- **Title:** `Live raffle card from Google Sheet`
- **Intent:** Show live raffle + 50/50 totals on the page during the event, read from the staff's Google Sheet, so we don't hand-update the site.
- **Acceptance criteria:** Totals update within ~30s of a sheet edit · Totals shown as big figures · A toggle to hide the itemized list · Reads the sheet privately (not a public CSV).
- **Verification checklist:** Edit a sheet cell, confirm the card moves within 30s · Confirm 50/50 total equals "Total Raised" · Confirm the sheet isn't publicly exposed.
- **Out of scope:** No editing the sheet from the site; display only.

Notice: every decision you made in chat this week is a line in that issue. Linear just makes it durable and checkable.

---

## How deep to go (and where you stopped)

There are three levels. You chose **(b)**.

| Level | What it means | Your call |
|---|---|---|
| (a) Tracking only | Linear is a to-do list; specs still live in chat | — |
| **(b) Spec source** | **Issues are the brief Claude reads first; acceptance + verification live there** | **✅ You're here** |
| (c) Drive the loop | Claude pulls its own next issue, builds, and reports back with less hand-holding | Later, once (b) is a habit |

You can graduate to (c) whenever (b) feels natural. No need to rush it.

---

## Gotchas (plain English)

- **The MCP connection is per-machine.** If you switch computers, you reconnect Linear once. (Automated/headless sessions may not have it — that's normal.)
- **Keep issues small.** One issue = one shippable thing. "Build the whole analytics system" is too big; "Add the per-page table to the analytics dashboard" is right.
- **The verification checklist is not optional.** It's the part that makes Linear worth it for you specifically. An issue with no verification section is just a sticky note.
- **You don't have to use Linear for everything.** Typos, quick tweaks, exploration — stay in chat. Linear is for the consequential, multi-step builds where a written finish line pays off.

---

## TL;DR
1. Write the work as a Linear issue using the **Intent / Acceptance / Verification** template.
2. Tell me "build RAD-XX." I read it, align with you, build, and link commits to it.
3. Before "done," I show you the diff and we run the verification checklist against real data.
4. The issue becomes a permanent record of what was built and how we proved it.

That's your "context engineering, engineered." Say *"connect Linear"* when you're ready and I'll get the connector wired up.
