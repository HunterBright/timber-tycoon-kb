---
name: three-level-analysis-system
description: Tasks classified by analysis depth — Level 1 (full analysis + accept, architectural), Level 2 (brief justification + accept, additive), Level 3 (immediate, single-value). Prevents over-thinking simple and under-thinking complex.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/claude-code
  tags: [claude-code, analysis, workflow, decision-process]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [claude-code-projects]
---

# Three-Level Analysis System

## The three levels

### Level 1 — Full analysis + explicit acceptance required (5-10 min thinking)

**When:**
- Architectural changes (new manager, new pattern, new system)
- Refactors touching 3+ files
- Anything affecting save format (ISaveable migration)
- New features with non-obvious side effects
- Open questions ("what do you think?", "how should we approach this?")

**What Code does:**
1. Analysis (problem context, which systems are affected)
2. 2-3 options with pros/cons
3. Recommendation with rationale
4. "Do you accept? Y/N" — **STOP, wait for answer**
5. Only implement after explicit "ok / yes / zatwierdzam"

### Level 2 — Brief justification + acceptance (1-2 min, 1-2 sentences)

**When:**
- New feature that follows existing pattern (new tree species, new product type)
- Bug fix with clear root cause
- New file that clearly belongs to established architecture

**What Code does:**
1. One sentence: "I'll use pattern X because it matches how we handle Y"
2. "Confirm?" — wait for answer
3. Implement after confirmation

### Level 3 — Immediate implementation (no analysis)

**When:**
- Single value change in existing ScriptableObject
- Variable rename
- Removing a single component
- Typos, formatting, comments

**What Code does:** Implements immediately. Short "done" report after.

## Escalation rule

**When uncertain which level:** always choose the higher level (more caution). The cost of an unnecessary Level 1 analysis is 5 minutes. The cost of a Level 3 that should have been Level 1 can be hours of debugging.

## Time budget

Level 1: allocate full attention (read all related files, check architecture docs).
Level 2: 1-2 file reads max.
Level 3: implement in one move.

See also: [[clear-vs-compact-decision-rules]], [[context-degradation-threshold]], [[quad-backtick-claude-code-prompt-format]]
