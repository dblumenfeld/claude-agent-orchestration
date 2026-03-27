---
description: Execute an approved plan step by step
argument-hint: [path-to-plan]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch
---

Read the plan at: $ARGUMENTS

Implement it step by step.

## Rules
1. Execute each step in order. After completing a step, briefly confirm what
   was done before moving to the next.
2. If a step's instructions are ambiguous, read the relevant source files to
   resolve the ambiguity rather than asking.
3. If a step fails, diagnose and fix before proceeding. Do not skip steps.
4. Run any verification/test commands specified in the plan after each phase.
5. At the end, run the plan's full verification section and report results.
