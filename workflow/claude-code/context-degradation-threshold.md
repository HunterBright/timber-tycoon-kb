---
name: context-degradation-threshold
description: Claude Code performance degrades visibly at 20-40% context remaining. Above 40% = full capability. Below 20% = unreliable output. Plan /clear before entering the danger zone.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/claude-code
  tags: [claude-code, context-window, prompt-strategy]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [claude-code-projects]
---

# Context Degradation Threshold (20-40% Remaining)

## What happens

Claude Code's output quality degrades visibly when context window falls below 40% remaining. Below 20% = unreliable. The degradation is subtle at first, then sudden.

**Indicators of degradation:**
- Re-reads files it already has in context
- Asks about previously stated facts ("what was the file path again?")
- Suggests solutions that contradict prior decisions made in the same session
- Misses simple edits ("forgot to remove the old block I replaced")
- Implements features that were already implemented earlier in the session

## Rule

**At 40% remaining:** Evaluate whether to `/clear` soon. Write current findings to disk (checkpoint file, task notes) before the decision point passes.

**Below 20%:** Stop work. Run `/compact` (to compress context and continue) or `/clear` (to start fresh). Any output below 20% carries risk of mistakes that are harder to spot than just running `/clear`.

## Why this happens

Model attention spreads across the entire context window. When context is full, early instructions (CLAUDE.md, architecture conventions, prior decisions) compete with recent messages for attention. The model doesn't "forget" but the signal-to-noise ratio drops. Recent context drowns out older critical constraints.

## Mitigation strategy

1. **Short sessions** per `/clear` cycle — single focused task per context
2. **Write findings to disk** before reaching the threshold (use checkpoint files)
3. **Front-load critical info** — put the most important constraints at TOP of prompt, not buried in the middle
4. **Recognize early** — "Code is being weirdly bad today" = usually context, not a model issue

## Common misdiagnosis

"Claude Code suddenly got worse." Usually context exhaustion, not a model regression. Check context indicator before blaming the model.

See also: [[clear-vs-compact-decision-rules]], [[three-level-analysis-system]], [[stop-hook-infinite-loop-risk]]
