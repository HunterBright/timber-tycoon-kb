---
name: skill-loading-on-demand-vs-reference
description: Skills via @reference (loaded immediately, costs context upfront) vs on-demand (Code searches when relevant, saves context budget). Use @ for small/always-needed, on-demand for large/episodic.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/mcp-tools
  tags: [claude-code, skills, context-budget, on-demand]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [claude-code-projects]
---

# Skill Loading: @Reference vs On-Demand

## Two loading strategies

| Strategy | How | Cost | When to use |
|----------|-----|------|-------------|
| `@skillname` reference | Skill file loaded into context on prompt submission | Context tokens upfront, immediate availability | Small skills, definitely needed this session |
| On-demand | Code searches for skill when topic comes up (tool_search / file read) | Round-trip cost, saves upfront context | Large skills, uncertain relevance |

## Decision rules

### Use `@` (immediate) for:
- Skills **definitely** needed in current session (active development on that domain)
- **Small** skills (< 200 lines) — cheap to load
- Skills Code forgets it should read (e.g., project conventions it might miss)

Examples in TT: `@unity-conventions` always loaded (small, always relevant to any Unity work)

### Use on-demand for:
- **Large** skills (> 300 lines) — expensive to load if not used
- Skills with **episodic** relevance (needed only when working on Blender, or only for map work)
- **Multi-skill scenarios** where several skills might be needed — load only what's actually relevant

Examples in TT: `@blender-export` on-demand (large, only needed during model pipeline work); `@map-environment` on-demand (only when doing terrain work)

## Context budget impact

With 5 skills loaded at once:
- 5 × 300 lines × 4 tokens/line ≈ 6,000 tokens consumed upfront
- That's ~10-15% of context budget before any work is done

Selective loading: load 1-2 skills at a time → 2-3% overhead.

**20-30% context savings** if skill loading is optimized across a session.

## Skill design tip

Design skills to be small and focused. A 500-line skill that covers everything is always loaded or never loaded correctly. A 100-line skill covering one domain is fast to load on-demand and easy to decide when to use.

## In TT's CLAUDE.md

The skill loading rules in CLAUDE.md specify which skills to load per task domain. Code must follow these rules rather than loading all skills by default.

See also: [[context-degradation-threshold]], [[clear-vs-compact-decision-rules]], [[mcp-wildcard-permissions-format]]
