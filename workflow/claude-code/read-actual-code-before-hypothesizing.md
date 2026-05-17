---
type: lesson
project: timber-tycoon
suggested-category: workflow/claude-code
tags: [claude-code, debugging, workflow, anti-pattern-of-thought]
severity: high
time_lost: "entire debugging sessions"
date: 2026-05-17
status: draft
applies_to: ["claude-code-projects"]
---

# Read Actual Code Before Hypothesizing

## Problem
When debugging, Claude (in chat mode especially) tends to generate plausible-sounding hypotheses from symptoms alone. These hypotheses waste time because they're often wrong — the real cause is visible in the code, not guessable from the symptom.

## Root cause
Symptom-pattern matching produces confident but incorrect diagnoses. "Collider is probably the wrong size" → spends 30 minutes adjusting colliders → bug is actually a script overwriting the collider values at runtime. Visual evidence (screenshots) is particularly misleading — what's visible ≠ ground truth in Unity.

## Solution
**First action when debugging**: read the actual code involved. Not infer, not hypothesize — read.

Protocol:
1. Ask for / find the relevant file
2. Read it fully
3. THEN diagnose based on what the code actually does

Especially critical when Hunter provides screenshots — visual state is not the same as code state.

## What didn't work
Hypothesis-first debugging from symptoms or screenshots — confidently proposes wrong solution, wastes session.

## Transferability
Applies to any AI-assisted debugging workflow. The pattern is: tool use (read code) before language model inference (hypothesize). Code is ground truth; model reasoning is a shortcut that often leads astray.

## Related
- [3-level analysis system](three-level-analysis-system.md)
- [Context degradation threshold](context-degradation-threshold.md)
