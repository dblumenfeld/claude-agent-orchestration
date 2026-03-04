# Plan & Review — Adversarial Planning Workflow for Claude Code

An automated spec-to-plan workflow using Claude Code Agent Teams. A Planner agent generates an implementation plan from a specification document. An independent Critic agent reviews it. They loop until the Critic approves (each loop spawns a fresh Critic with no context of previous reviews). You get the final plan to sign off before any code is written.

## Why This Works

The core insight is **context isolation**. When the same session that writes a plan also reviews it, the reviewer is polluted by the author's reasoning — it tends to rubber-stamp its own work. By spawning a fresh Critic each review round with zero context from the planning session, you get genuinely independent feedback. This is sometimes called adversarial review or the four-eyes principle.

The workflow encodes a pattern you'd otherwise do manually:
1. Write plan in Session A
2. Open fresh Session B, paste plan, get critique
3. Copy critique back to Session A, revise
4. Clear Session B, repeat

This repo automates steps 2–4.

---

## Prerequisites

- [Claude Code](https://code.claude.com) installed
- Claude Max plan (Agent Teams uses significant tokens across multiple sessions)

---

## Setup

### 1. Enable Agent Teams

Add to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Reload it:

```bash
source ~/.zshrc
```

### 2. Copy the `.claude/` directory into your project

```
your-project/
  .claude/
    commands/
      plan-and-review.md    ← slash command (the orchestrator)
    agents/
      planner.md            ← planner agent definition
      critic.md             ← critic agent definition
```

You can find the slash command and agent definitions in this repo:

[Plan and Review Command](plan-and-review.md)
[Planner Agent Definition](planner.md)
[Critic Agent Definition](critic.md)

You can either clone this repo and copy `.claude/` into your project, or create the files manually using the contents below.

---

## Usage

Write your spec in a markdown file and save it locally.

Open Claude Code in your project root, then:

```
/plan-and-review path/to/your-spec.md
```

Example:

```
/plan-and-review docs/feature-spec.md
```

The lead agent will:
1. Spawn the Planner and pass it your spec
2. Spawn a fresh Critic for each review round
3. Route feedback back to the Planner until the Critic approves
4. Present the final approved plan to you for sign-off

The draft plan is written to `.claude/plan-draft.md` and survives across rounds since it lives on disk, not in any agent's context.

---

## Implementing the Approved Plan

Once you approve the plan, start a fresh session to implement it:

```bash
# Option A: clear context in the current session
/clear

# Option B: open a new Claude Code session entirely (recommended — harder reset)
```

Then type the prompt:

```
Read `.claude/plan-draft.md` and implement it step by step.
Confirm each step before moving to the next.
```

If you want a slash command for this too, create `.claude/commands/implement-plan.md`:

```markdown
---
description: Implement the approved plan from .claude/plan-draft.md
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

Read `.claude/plan-draft.md` and implement it step by step.
After completing each step, confirm it before moving to the next.
```

Then your full end-to-end workflow is:

```
/plan-and-review docs/my-spec.md   ← plan + review loop
# approve the plan
/clear
/implement-plan                     ← execution
```

---

## How It Works

```
You
 └── Lead (orchestrator)
      ├── Planner (stays alive, revises plan-draft.md each round)
      └── Critic-1 (spawned fresh, reviews, shut down)
      └── Critic-2 (spawned fresh, reviews, shut down)
      └── Critic-N (spawned fresh, reviews, shut down → APPROVE)
           └── Lead presents final plan to you
```

The Critic is re-spawned each round rather than cleared in place. This guarantees a completely clean context for every review — the Critic only ever sees the current version of the plan, with no memory of previous rounds or the planning process.

The plan file on disk (`.claude/plan-draft.md`) is the only shared state between rounds.

---

## Token Cost

Agent Teams use significantly more tokens than a single session. Each teammate is a full independent Claude instance. For a 3-round loop you're looking at roughly 5–8x the token cost of a single session doing the same work.

This is worthwhile for non-trivial plans where the cost of implementing a flawed plan far exceeds the cost of getting it right first.

---

## Background

For more about Claude Code agent teams, please refer to the official documentation: https://code.claude.com/docs/en/agent-teams

This pattern is sometimes called **adversarial review** or **multi-agent debate** in the AI research literature. The core property — an independent reviewer with no context from the planning session — maps to the **four-eyes principle** used in software engineering and regulated industries: any significant work must pass through a genuinely independent second set of eyes before proceeding.
