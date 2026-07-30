---
title: MCP Wildcard Permissions Format
type: lesson
status: needs-reproduction
confidence: low
verified: '2026-07-30'
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- claude-code
- mcp
- permissions
- configuration
applies_to:
- claude-code-projects
source: zweryfikowane reprodukcja i dokumentacja 2026-07-30, patrz AUDYT-SPORNYCH-WPISOW
description: MCP permissions in .claude/settings.json use double-underscore format mcp__{server}__* (wildcard) or mcp__{server}__{tool} (specific). Wrong format = server silently fails to connect.
severity: high
suggested-category: workflow/mcp-tools
name: mcp-wildcard-permissions-format
audit_verdict: DO SPRAWDZENIA
---

# MCP Wildcard Permissions Format

> [!warning] Ten wpis zostal zweryfikowany 2026-07-30 i werdykt brzmi: **DO SPRAWDZENIA**
>
> Pelne uzasadnienie, dowody i proponowane poprawki: [[AUDYT-SPORNYCH-WPISOW]].
> Tresc ponizej NIE zostala jeszcze przepisana - czytaj ja z ta uwaga.


## Rule

MCP tool permissions in `.claude/settings.json` use double-underscore format:

```json
{
  "permissions": {
    "allowed_tools": [
      "mcp__blender-mcp__*",
      "mcp__coplay-mcp__*",
      "mcp__elevenlabs__*",
      "mcp__screen-capture-mcp__*"
    ]
  }
}
```

**Wildcard:** `mcp__{server}__*` - allows all tools from that MCP server.
**Specific:** `mcp__{server}__{tool_name}` - allows only that tool.

## Critical: double underscore, not single

`mcp__blender-mcp__*` ✅ (double underscore separator)
`mcp_blender-mcp_*` ❌ (single underscore - silently fails)
`mcp__blender-mcp__create_object` ✅ (specific tool, double underscore)

The double underscore is MCP protocol convention. Single underscore is reserved for other uses in the protocol.

## Failure mode

Wrong format = MCP server doesn't connect. No error message in console. Tool calls return "permission denied" or time out silently. Very confusing to debug because the failure doesn't indicate what's wrong.

## TT server list

| Server | Wildcard pattern |
|--------|-----------------|
| Blender MCP | `mcp__blender-mcp__*` |
| Coplay Unity | `mcp__coplay-mcp__*` |
| ElevenLabs | `mcp__elevenlabs__*` |
| Screen capture | `mcp__screen-capture-mcp__*` |

## History

Pre-April 2026 TT config used inconsistent formats (single underscore, no prefix). April 2026 audit found several MCP servers silently failing. Fix: standardize all to `mcp__{server}__*`. Restart Claude Code after settings changes.

See also: [[skill-loading-on-demand-vs-reference]], [[stop-hook-infinite-loop-risk]]
