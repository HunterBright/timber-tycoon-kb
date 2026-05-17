# Hunter's Knowledge Base

Long-term, cross-project knowledge base for Unity solo dev work.
Survives projects. Grows with each session.

## What lives here vs. what lives in memory

**KB (this folder)** — transferable cross-project wisdom:
- Unity/Blender bugs and gotchas (engine-level)
- Validated workflow patterns (asset pipeline, Claude Code conventions)
- Architecture Decision Records with trade-offs
- Anti-patterns: what NOT to do

**Project memory** (lives at C:\Users\<user>\.claude\projects\<project>\memory\):
- Project-specific feedback (colors, positions, design preferences)
- Current sprint state
- Per-project decisions still in flux
- Machine-local, not in git

Different scope, different lifetime, different storage. Both intentional.

## Structure
- engine/ — Unity/Blender/URP wisdom (transferable across all projects)
- genre/ — Genre-specific patterns (tycoon, survival, roguelike, pvp)
- workflow/ — Meta-tooling: Claude Code, MCP, asset pipeline
- projects/ — Per-project indexes (links only, no content)
- _inbox/ — Drafts awaiting weekly review (Claude writes here)
- _archive/ — Superseded entries (kept for history)
- templates/ — Entry templates (lesson/pattern/decision/anti-pattern)

## Workflow
1. During sessions, Claude Code auto-writes drafts to _inbox/
2. Sunday evening: /kb-review — batch approve/edit/reject drafts
3. New projects: junction kb\ → this folder, instant access

## How Claude Code uses this
Global CLAUDE.md at C:\Users\<user>\.claude\CLAUDE.md instructs Claude to:
- Read MOC.md when task is non-trivial and likely has prior art
- Auto-write drafts when triggers fire (see global CLAUDE.md)
- Not interrupt flow with "should I save?" prompts
