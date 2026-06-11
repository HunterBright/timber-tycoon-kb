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

## Case study: kinematic reverse rotation (2026-05-22)

**Symptom:** NPC car visually slid sideways along the reverse-parking arc instead of pointing along it.

**Confident-but-wrong hypothesis:** the architect assumed the bug was the Blender FBX forward-axis quirk (`-transform.right`) combined with a `Quaternion.LookRotation` aligning +Z to the path tangent.

**What the code actually showed:** there was no `LookRotation` at all. Body rotation was a blind time-based `Quaternion.Slerp(startRot, endRot, t)` that ignored the path tangent entirely. The tangent was computed but used only for wheel steering.

**Lesson reinforced:** even a strong, specific hypothesis from someone who knows the codebase well was wrong. Reading the actual coroutine body before prescribing a fix is what caught it. *(Note: the subsequent fix attempts themselves then violated this rule by guessing cross-product directions theoretically instead of logging real runtime vectors — a second reminder that vector orientations must be confirmed empirically, not derived on paper.)*

## Related
- [3-level analysis system](three-level-analysis-system.md)
- [Context degradation threshold](context-degradation-threshold.md)
