---
title: Unity MCP "Connection revoked" is a misleading error - the real causes are capacity limit and missing AI entitlement
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-21'
project: Kerf - Sawmill Tycoon
tags:
- unity
- mcp
- unity-ai
- licensing
- entitlements
- diagnostics
- misleading-error
applies_to:
- unity-6000.5
- com.unity.ai.assistant
source: ''
severity: high
time_lost: ~40 min
promoted: '2026-07-30'
---

# Unity MCP "Connection revoked" is a misleading error - the real causes are capacity limit and missing AI entitlement

## Problem

Every `unity-mcp` tool call fails instantly with:

```
Connection revoked. Go to Unity Editor > Project Settings > AI > Unity MCP to change approval.
```

The message points the user at an approval toggle. Following it wastes time:

- the settings page is actually named **"Unity MCP Server"** (path `Project/AI/Unity MCP Server`), not "Unity MCP"
- even when found, there is **no approval to change** - the user never denied anything
- reconnecting the MCP client (`/mcp` -> Reconnect) does not help
- closing competing clients does not necessarily help either

## Root cause

`Bridge.cs` returns that one string for **every** `ConnectionApprovalState.Denied`, regardless of why the state
became Denied. There are at least four distinct paths to Denied (Bridge.cs ~1140-1250):

1. connection policy `allowed == false`
2. **capacity limit** - `ConnectionCensus.TryReserveDirect` refuses (`ValidationStatus.CapacityLimit`)
3. batch mode with `autoApproveInBatchMode == false`
4. a genuine previous user rejection (`ValidationStatus.Rejected`)

Only case 4 matches what the error text claims.

The capacity cap is **not a project setting**. It comes from the Unity licensing entitlement
`com.unity.editor.ai` via `EditorConnectionLimitProvider` -> `LicensingUtility.HasEntitlementsExtended`.
`Data.cs` defaults `AllowedMcpConnections = result?.AllowedMcpConnections ?? 0` - so **no entitlement means a
cap of zero and every MCP connection is denied**, with the same "revoked" wording.

Observed progression on one machine: 2026-07-17 cap was `1/1` (one slot, taken by a second client);
2026-07-21 the plan carried no AI entitlement at all and the reason string became
`Your Unity plan doesn't include MCP connections.`

## Solution

Do not trust the error string. Read the two files that hold the truth:

1. **Connection registry** - `Library/AI.MCP/connections-v2.asset` (ScriptableSingleton, YAML).
   Each record has `Status:` as an int of `ValidationStatus`:
   `0=Pending, 1=Accepted, 2=Rejected, 3=Warning, 4=CapacityLimit`,
   plus `ValidationReason` in plain English and `TimestampTicks` (.NET ticks -> `[datetime]::new($ticks)`).
   A wall of `Status: 4` means capacity, not rejection. `Status: 2` would mean the user really did reject.

2. **Licensing client log** - `%LOCALAPPDATA%\Unity\Unity.Licensing.Client.log`.
   Look for `EntitlementDetailsResponse`. The line
   `[404]: Found 0 entitlement groups and 0 free entitlements matching requested entitlement ids`
   proves the account has no AI entitlement, even while
   `Got access token` / `Successfully updated license file` prove the user IS signed in.

Then act on the actual cause:
- capacity reached, cap >= 1 -> close the other MCP client (a second Claude/Cursor/Codex instance holds the slot)
  and force a **new** connection; the Denied state lives per-transport in memory, so the old transport never recovers
- cap == 0 (no entitlement) -> nothing local fixes it; the plan itself lacks Unity AI

## What didn't work

- Hunting for the approval toggle in Project Settings (wrong mental model, and the page name differs from the error text)
- `/mcp` -> Reconnect (creates a new transport, which hits the same cap and is denied again)
- Assuming "revoked after script recompile", a plausible-sounding folk explanation carried over from an earlier
  session note; it fit the symptom and was wrong. The registry file disproved it in one query.
- Closing the competing client alone - necessary when the cap is 1, useless when the cap is 0

## Transferability

Any Unity 6 project using `com.unity.ai.assistant` and an external MCP client (Claude Code, Cursor, Codex,
Claude Desktop) hits this. The lesson generalises past Unity: **when a gate reports one reason and the code has
four paths into that gate, read the persisted state, not the message**. Unity conveniently persists both the
per-connection verdict and the licensing answer to disk, so the diagnosis is a two-file read, not guesswork.

Second, transferable planning point: **entitlement-gated tooling can disappear between sessions without any
local change**. Tool availability recorded in project notes needs a date and a re-check, not a permanent claim.

## Related
- [[project_unity_mcp_tooling_notes_2026-07-17]] (recorded "unity-mcp works, Coplay 401"; the first half expired within four days)
