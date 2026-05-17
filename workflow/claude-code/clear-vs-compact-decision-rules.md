---
name: clear-vs-compact-decision-rules
description: /clear = full reset (switching topics). /compact = compress current context (continue same task). 2 terminals max parallel. Decision rules for when to use each.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/claude-code
  tags: [claude-code, context-management, clear, compact]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [claude-code-projects]
---

# /clear vs /compact — Decision Rules

## The two tools

| Command | Effect | Use when |
|---------|--------|----------|
| `/clear` | Full context reset | New unrelated task, previous task complete, context heavily degraded |
| `/compact` | Summarize current context into smaller footprint | Mid-task need for more headroom, don't want to lose current state |

## Decision rules

### Use `/clear` when:
- Starting a task in a different system (e.g., just finished NPC traffic, now doing UI)
- Previous task is complete and committed
- Context degraded below 20% (see [[context-degradation-threshold]])
- Too many iteration cycles on the same thing — context is "noisy" with obsolete versions

### Use `/compact` when:
- Mid-task, approaching completion, just need a bit more headroom
- Current state is important to preserve (half-written implementation)
- Same system, new subtask within it

## Two terminals max

Running 2 terminals in parallel:
- Terminal 1: primary task
- Terminal 2: emergency diagnostic (e.g., "check this file while main terminal runs build")

Running 3+: cognitive load too high. Easy to lose track of which terminal has which context, which decisions were made where. Mistakes cross-contaminate.

## Hunter's convention

Prompt format signals context decision:
```
/clear: TAK — fresh start, nowy temat
```
or
```
/clear: NIE — kontynuacja [nazwa systemu]
```

This appears at top of every new prompt. Claude Code reads it before doing anything else.

## Why context management matters

Context management is the difference between a productive session and "Code is confused" session. Spending 30 seconds on `/clear` vs `/compact` decision saves 20 minutes of degraded output.

See also: [[context-degradation-threshold]], [[stop-hook-infinite-loop-risk]], [[three-level-analysis-system]]
