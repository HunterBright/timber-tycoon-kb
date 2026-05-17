---
name: console-collapse-loop-suffix
description: Unity Console Collapse mode merges identical log strings. In loops, add unique [#{i}] suffix per iteration — all logs visible even with Collapse ON.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/claude-code
  tags: [unity, debug-log, console, collapse, claude-code]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Console Collapse + Loop Suffix Convention

## What happened

Unity Console "Collapse" mode (default ON) merges logs that have identical string content. A loop printing 10 log entries shows as a single entry with "×10" counter — the actual per-iteration data is invisible.

Symptom: "Why is my loop only processing item 3?" — it isn't. Console is hiding the other 9.

## Fix

Include a unique suffix in every loop log:

```csharp
// WRONG — Collapse merges all of these into one entry
for (int i = 0; i < items.Count; i++) {
    Debug.Log($"Processing {items[i].name}"); // all merge as "Processing item"
}

// CORRECT — each log is unique, all visible
for (int i = 0; i < items.Count; i++) {
    Debug.Log($"[#{i}] Processing {items[i].name}");
}
```

The `[#{i}]` prefix ensures every log string is unique → Collapse cannot merge them.

## Convention in Timber Tycoon

**All loop logs include `[#{i}]` or another unique marker.** This is enforced by Claude Code on all generated loop debugging code.

For non-indexed loops (foreach):
```csharp
int idx = 0;
foreach (var item in items) {
    Debug.Log($"[#{idx++}] {item.name}");
}
```

## Alternative

Turn off Collapse in Console toolbar (small icon, 4th from left). Works but easy to forget to turn back on — then truly identical errors stop being grouped, which IS useful.

Convention is better: unique suffix is self-documenting, works regardless of Console mode.

## Time cost

5-minute waste prevented: "loop not running" debugging when logs were actually just collapsed. Convention costs 0 extra seconds when established.

See also: [[context-degradation-threshold]], [[three-level-analysis-system]]
