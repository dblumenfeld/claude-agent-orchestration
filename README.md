# Plan & Review — Adversarial Planning Workflow for Claude Code

An automated spec-to-plan workflow using Claude Code Agent Teams. A Planner agent generates an implementation plan from a specification document. An adversarial Critic agent stress-tests it. They loop — with the Critic verifying claims against the actual codebase and auditing prior feedback — until the plan meets a strict approval checklist. You get the final plan to sign off before any code is written.

## Why This Works

The core insight is **adversarial tension with accountability**. The Critic is designed to find problems, not approve plans. It must verify claims by reading actual source files, audit whether prior feedback was genuinely resolved (not just acknowledged), and meet a strict multi-point checklist before it can approve. The Planner, in turn, must address every issue individually — either fixing it or pushing back with reasoning. This creates a productive dialectic where neither side can hand-wave.

The workflow enforces a minimum of 4 review rounds. Each new Critic receives the full history of prior feedback, so it can build on previous concerns rather than starting from scratch. This prevents the common failure mode where a fresh reviewer misses issues that were already identified and supposedly fixed.

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

### 2. Install globally

The commands and agents install to `~/.claude/` so they're available in every project.

**Option A: Automatic (from any Claude Code session)**

```
/setup-orchestration
```

This pulls the latest files from GitHub and installs them globally.

**Option B: Manual**

Clone this repo and copy the files:

```bash
git clone https://github.com/dblumenfeld/claude-agent-orchestration.git
cd claude-agent-orchestration
mkdir -p ~/.claude/agents ~/.claude/commands
cp planner.md ~/.claude/agents/
cp critic.md ~/.claude/agents/
cp plan-and-review.md ~/.claude/commands/
cp review-plan.md ~/.claude/commands/
```

Your global layout should be:

```
~/.claude/
  agents/
    planner.md            ← planner agent definition
    critic.md             ← adversarial critic agent definition
  commands/
    plan-and-review.md    ← orchestrator slash command
    review-plan.md        ← standalone review slash command
    setup-orchestration.md ← self-update command
```

---

## Usage

### Full plan-and-review loop

Write your spec in a markdown file and save it locally. Open Claude Code in your project root, then:

```
/plan-and-review path/to/your-spec.md
```

The orchestrator will:
1. Spawn the Planner and pass it your spec
2. Spawn a fresh Critic for each review round, passing the current plan and all prior feedback history
3. Enforce a minimum of 4 rounds — early approvals are converted to revision requests
4. Route feedback back to the Planner, requiring per-issue responses
5. Accept the Critic's approval once the strict checklist is met (round 4+)
6. Present the final plan to you for sign-off

### Standalone review

To review any plan without the full loop:

```
/review-plan path/to/plan.md
```

### Updating

To pull the latest versions from GitHub:

```
/setup-orchestration
```

---

## How It Works

```
You
 └── Lead (orchestrator)
      ├── Planner (stays alive, revises plan-draft.md each round)
      └── Critic-1 (reviews, shut down — feedback archived)
      └── Critic-2 (reviews with round 1 history, shut down)
      └── Critic-3 (reviews with rounds 1-2 history, shut down)
      └── Critic-N (reviews with full history → APPROVE if checklist met)
           └── Lead presents final plan to you
```

Each Critic is spawned fresh but receives the **full critique history** from all prior rounds. This means Critic-3 knows what Critic-1 and Critic-2 found, and can verify those issues were actually resolved rather than just reworded.

The Planner maintains a **Revision Log** at the end of the plan, tracking what changed each round and what was pushed back on. This creates an audit trail the Critic can reference.

The plan file on disk (`.claude/plan-draft.md`) is the shared state between rounds.

### Critic approval checklist

The Critic can only issue APPROVE when ALL of these are true:
- Every spec requirement has a corresponding plan step detailed enough to implement without guesswork
- At least 5 codebase claims verified by reading actual files with no contradictions found
- No critical or high-severity issues remain
- All prior critic feedback substantively addressed (not just reworded)
- Error handling and edge cases explicitly covered for I/O, parsing, and external calls
- Plan includes a verification/testing strategy for each phase

### Typical convergence

| Round | What happens |
|-------|-------------|
| 1 | Critic finds 5-8 issues (codebase checks reveal wrong assumptions, missing error handling, spec gaps). Planner revises, pushes back on 1-2. |
| 2 | Critic audits prior feedback, finds some not fully fixed, discovers new issues from deeper reading. Planner revises. |
| 3 | Critic confirms most prior issues fixed, finds new edge cases and testing gaps. Planner revises. |
| 4+ | First round eligible for APPROVE. Strict checklist means approval typically lands rounds 5-7. |

---

## Implementing the Approved Plan

Once you approve the plan, start a fresh session to implement it:

```bash
# Option A: clear context in the current session
/clear

# Option B: open a new Claude Code session entirely (recommended — harder reset)
```

Then:

```
Read `.claude/plan-draft.md` and implement it step by step.
Confirm each step before moving to the next.
```

---

## Token Cost

Agent Teams use significantly more tokens than a single session. Each teammate is a full independent Claude instance. With the minimum 4-round loop and typical 5-7 round convergence, expect roughly 10-15x the token cost of a single session.

This is worthwhile for non-trivial plans where the cost of implementing a flawed plan far exceeds the cost of getting it right first.

---

## Background

For more about Claude Code agent teams, refer to the official documentation: https://code.claude.com/docs/en/agent-teams
