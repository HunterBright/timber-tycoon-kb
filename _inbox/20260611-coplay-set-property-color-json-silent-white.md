---
type: lesson
project: Timber Tycoon
suggested-category: workflow/mcp-tools
tags: [coplay, mcp, set_property, color, unity, silent-failure]
date: 2026-06-11
status: draft
---

# Coplay set_property: Color fields need comma-separated r,g,b,a — JSON silently writes white

## Problem
Setting a `Color` field via Coplay MCP `set_property` with a JSON-style value (e.g. `{"r":0.2,"g":0.4,"b":0.1,"a":1}`) **reports success** but actually writes **white** (1,1,1,1). No error, no warning — the only symptom is the object rendering white later.

## Fix
Pass the value as a plain comma-separated string: `0.2,0.4,0.1,1` (r,g,b,a). This parses correctly and writes the intended color.

## Detection rule
Any Coplay set_property call on a Color that "succeeded" but the object renders white → re-send with the comma format and re-verify visually. Treat every reported success on Color fields as unverified until seen rendered.

## Transferability
Any project driving Unity through Coplay MCP. General family: MCP tools that report success while silently coercing/defaulting an unparseable value — verify visually, not by tool status (see [[debugging-search-first-trust-render-check-upstream]] Rule 2).
