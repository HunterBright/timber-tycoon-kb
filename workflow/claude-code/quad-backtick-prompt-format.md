---
title: Quad-Backtick Claude Code Prompt Format
type: lesson
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- claude-code
- prompt-format
- syntax
- convention
applies_to:
- claude-code-projects
source: ''
description: Hunter's standard prompt format - wrapped in 4 backticks, starts with
severity: high
suggested-category: workflow/claude-code
name: quad-backtick-claude-code-prompt-format
---

# Quad-Backtick Claude Code Prompt Format

## Format specification

````
# TASK: Short descriptive title

**File:** `Assets/Project/Scripts/SomePath.cs`

Find:
```
exact text from file, verbatim
```

Replace with:
```
new text, verbatim
```

Do NOT modify any other files.
````

**Outer wrapper:** 4 backticks (````). This allows nested triple-backtick code blocks inside without breaking markdown parsing.

## Critical: /clear is a SEPARATE block

Hunter **never** puts `/clear` in the same block as the task. They are submitted as two separate messages:

First message:
```
/clear: TAK - fresh start, new topic
```

Second message (the actual task, in quad-backtick format):
````
# TASK: ...
````

**Why:** Claude Code parses messages sequentially. If `/clear` and the task are in the same block, Code processes the task in the OLD context (before the clear took effect). This is a common mistake when learning the workflow.

## Find/Replace blocks

For exact edits, the `Find:` / `Replace with:` block eliminates ambiguity:
- Code knows exactly which text to match
- No guessing about whitespace or indentation
- If `Find:` text isn't found, Code reports error rather than making wrong edit

## Scope boundary

Every task ends with explicit scope boundary:
```
Do NOT modify any other files.
```
or
```
Only modify: Assets/Project/Scripts/FileName.cs
```

Without this, Code may "helpfully" edit related files that weren't part of the task.

## Why this format works

Consistency = reliable parsing. Code handles this format without ambiguity. Variations in format = variations in how Code interprets the task. One canonical format → fewer parse errors → fewer wrong edits.

See also: [[three-level-analysis-system]], [[clear-vs-compact-decision-rules]], [[iterative-checkpoint-workflow]]
