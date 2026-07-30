---
title: Stop Hook Infinite Loop Risk
type: lesson
status: needs-reproduction
confidence: low
verified: '2026-07-30'
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- claude-code
- hooks
- configuration
- infinite-loop
applies_to:
- claude-code-projects
source: zweryfikowane reprodukcja i dokumentacja 2026-07-30, patrz AUDYT-SPORNYCH-WPISOW
description: Claude Code Stop hook (auto-continue on context full) was tested and removed. Caused infinite loops on stuck tasks. Manual /continue is safer - human evaluates progress each cycle.
severity: high
suggested-category: workflow/claude-code
name: stop-hook-infinite-loop-risk
audit_verdict: DO SPRAWDZENIA
---

# Stop Hook Infinite Loop Risk

> [!warning] Ten wpis zostal zweryfikowany 2026-07-30 i werdykt brzmi: **DO SPRAWDZENIA**
>
> Pelne uzasadnienie, dowody i proponowane poprawki: [[AUDYT-SPORNYCH-WPISOW]].
> Tresc ponizej NIE zostala jeszcze przepisana - czytaj ja z ta uwaga.


## What happened

Claude Code Stop hook (configured in `.claude/settings.json`) triggers a script when context capacity is reached. The intention: auto-continue the task without manual intervention.

**Actual behavior:**
1. Code reaches context cap
2. Stop hook triggers new Code instance
3. New instance also hits context cap (same task, same problem)
4. Hook triggers again
5. Infinite loop - ran overnight, accumulated Claude API credits

Worst case: Code stuck on an unsolvable problem (race condition that doesn't reliably reproduce). Hook spins indefinitely because Code never makes progress that would satisfy the stop condition.

## Decision

Stop hook removed from TT's `.claude/settings.json`.

**Replacement:** Hunter manually invokes `/continue` or `/clear` after evaluating whether progress was made. Human evaluation = stop condition.

## Why auto-continue is dangerous

Automation without a stop condition = potential infinite loop. The Stop hook has no built-in "bail out if N iterations without progress" logic. Any task where Code can't make progress becomes infinite.

## General lesson

For any automated loop (hooks, cron jobs, CI scripts): always define the stop condition BEFORE implementing the loop. "Run until done" is not a stop condition - "done" must be detectable and achievable.

## Configuration note

If Stop hook is needed for a specific long-running task:
- Limit to max N iterations: track iteration count in a temp file
- Bail out explicitly if no git diff between iterations (no progress detected)
- Always have a timeout (wall-clock or iteration count)

See also: [[context-degradation-threshold]], [[clear-vs-compact-decision-rules]]
