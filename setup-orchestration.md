---
description: Install shared agent orchestration files from GitHub into the current project
allowed-tools: Bash, Read, Glob
---

Pull the latest agent orchestration files from
`dblumenfeld/claude-agent-orchestration` on GitHub and install them into the
current project's `.claude/` directory.

## File Mapping

The repo has a flat structure. Map files to local paths as follows:

| Repo file              | Local path                              |
|------------------------|-----------------------------------------|
| `planner.md`           | `.claude/agents/planner.md`             |
| `critic.md`            | `.claude/agents/critic.md`              |
| `plan-and-review.md`   | `.claude/commands/plan-and-review.md`   |
| `review-plan.md`       | `.claude/commands/review-plan.md`       |

## Steps

1. Create `.claude/agents/` and `.claude/commands/` directories if they don't exist
2. For each file in the mapping above, fetch it with:
   ```
   curl -sL https://raw.githubusercontent.com/dblumenfeld/claude-agent-orchestration/main/{repo_file}
   ```
3. Before overwriting, check if the local file already exists:
   - If it doesn't exist: write the fetched content and report "installed"
   - If it exists and is identical: report "already up to date"
   - If it exists and differs: overwrite and report "updated"
4. Print a summary of what was installed, updated, or already current
