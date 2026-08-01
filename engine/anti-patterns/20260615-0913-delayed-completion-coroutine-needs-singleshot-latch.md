---
title: A delayed-completion coroutine that still reads input double-fires without a single-shot latch
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-06-15'
project: Kerf - Sawmill Tycoon
tags:
- unity
- coroutine
- input
- minigame
- state-management
- race-condition
- resource-accounting
applies_to: []
source: ''
promoted: '2026-07-30'
---

# A delayed-completion coroutine that still reads input double-fires without a single-shot latch

## Context
A common minigame pattern: when the win condition is reached, show a result for a
beat, then fire a completion callback after a short delay:

```csharp
if (currentHits >= hitsRequired)
    StartCoroutine(CompleteAfterDelay());   // waits 0.5s, then onComplete?.Invoke()
```

Concrete case: `SwingArcMinigameUI` (chopping minigame). `CompleteAfterDelay` waited
0.5s before setting `isActive = false` and invoking the callback. Meanwhile `Update()`
kept reading LMB → `HandleClick`, and the win condition (`currentHits >= hitsRequired`)
was still true (the hit counter is only reset on the *next* arm, not at completion).

## The anti-pattern
The completion is triggered by a condition that **stays true** during the delay
window, while input that re-tests that condition **keeps being processed**. Nothing
marks "completion already started." So every extra click during the delay starts
*another* `CompleteAfterDelay`, and each one invokes the completion callback.

The damage is in the callback, not the minigame:
- Callback consumed a resource + advanced an index per invocation → one double-clicked
  cut **burned two logs** and skipped a queued item.
- On a sibling code path where the resource was consumed up front instead, the extra
  invocations **produced free output** (firewood) with no extra cost.
- On the last item of a queue, a stale invocation re-entered after teardown cleared
  the queue → would have been an `IndexOutOfRangeException` without a bounds guard.

A logging trap made it worse: the consume helper logged "consumed" *unconditionally*
before checking whether anything was actually removed - so the console "proved"
consumption that never happened. (Separate lesson: a log line above the success check
is not evidence of the action.)

## Fix - a single-shot latch reset at arm time
```csharp
bool completing;

void HandleClick()
{
    if (completing) return;          // ignore all input once completion is queued
    ...
    if (currentHits >= hitsRequired)
    {
        completing = true;           // latch BEFORE starting the coroutine
        StartCoroutine(CompleteAfterDelay());
    }
}

void ArmCut()                        // called for every cut / every fresh run
{
    ...
    completing = false;              // deterministic reset on entry, not on consume
}
```

Resetting the latch in the arm method (which every entry path funnels through) makes
one fix cover every mode at once. Combine with: reset transient flags on **entry**,
never "set on the way in, clear on use" (see sibling lesson on one-shot flag bleed on
persistent singletons).

## Rule of thumb
If a completion fires from a condition that remains true during a delay, and input is
still polled during that delay, you have a double-fire waiting to happen. Latch
"completion started" the instant you queue it, gate the input handler on the latch,
and reset the latch at the next arm/entry. Put resource accounting behind the latch,
never behind the raw input event.

## How it was caught
The session (resource-burning) variant showed up as players losing extra logs on
rapid clicks; the legacy (free-output) variant was masked by Unity Console "Collapse"
folding the duplicate "+N firewood" lines. Disabling Collapse + an honest consume log
made the double-fire visible. Root cause confirmed straight from the source (counter
reset only in arm, `isActive` cleared only after the delay, no latch) rather than from
logs.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260614-1343-singleton-oneshot-flag-bleed|One-shot input flag on a persistent singleton bleeds across re-entries]] - wspolne: state-management, input, minigame
- [[20260717-1100-presave-flush-for-world-automation|Pre-save flush dla systemow automatyzacji mutujacych zapisywany swiat]] - wspolne: race-condition, coroutine
- [[20260724-1817-diegetic-button-overlap-steal|Powiekszone collidery klikania ciasnych guzikow 3D + wybor "pierwsze trafione pudelko" = sasiad kradnie klik]] - wspolne: input, minigame
<!-- /POWIAZANE:auto -->
