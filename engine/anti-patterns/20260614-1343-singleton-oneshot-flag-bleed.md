---
title: One-shot input flag on a persistent singleton bleeds across re-entries
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-06-14'
project: Kerf - Sawmill Tycoon
tags:
- unity
- singleton
- state-management
- input
- minigame
- refactor
- persistent-monobehaviour
applies_to: []
source: ''
promoted: '2026-07-30'
---

# One-shot input flag on a persistent singleton bleeds across re-entries

## Context
Persistent singleton MonoBehaviours (e.g. a minigame UI that lives for the whole
session via `Instance`) often add a one-shot guard like "swallow the click that
opened/armed this minigame so it doesn't count as the first action". The natural
first implementation sets the flag conditionally on entry and clears it only when
it is *consumed*.

Concrete case: `SwingArcMinigameUI` (chopping minigame) was refactored to support a
multi-cut "session" mode alongside the legacy single-cut mode. A field
`bool swallowNextClick` was set in the per-cut arm method **only when**
`sessionActive`, and reset to false **only** inside `HandleClick` when the click
was actually swallowed.

## The anti-pattern
```csharp
// arm a cut
if (sessionActive)
    swallowNextClick = true;   // set conditionally
...
// later, in click handler
if (swallowNextClick) { swallowNextClick = false; return; }  // cleared only on consume
```

Because the field lives on a **persistent singleton**, an unconsumed `true` survives
the whole minigame and bleeds into the *next, unrelated* entry:

1. Session starts → flag set true.
2. Session ends before any click consumes it (ESC/cancel, or the last armed cut
   was never clicked).
3. Teardown/end methods do **not** reset the flag.
4. A later **legacy** entry's arm path neither sets nor clears it → still true.
5. The player's first legitimate click in the legacy minigame is silently eaten -
   a lost input, no error, hard to repro.

The legacy path was "byte-for-byte unchanged" *in isolation*, but the shared
singleton state made it change once both paths coexisted.

## Fix
Make the flag **deterministic at every entry/arm**, not "set in one branch, cleared
on consume":
```csharp
// every arm explicitly assigns the correct value for THIS entry
swallowNextClick = sessionActive;   // true in session, false in legacy - always reset
```
Belt-and-suspenders alternative: also clear it in the end/cancel/teardown methods.
The point is that no code path may *assume* a prior path left the flag in a known
state.

## Rule of thumb
On a persistent singleton, treat every one-shot/transient flag as if a hostile
prior caller left it in the worst state. Reset transient state on **entry**
(deterministic assignment), not only on **consumption**. "Set on the way in,
clear on use" leaks whenever the use never happens.

## How it was caught
Adversarial multi-agent review (a dedicated "runtime-correctness skeptic" lens that
specifically hunted for cross-entry state bleed). A pure spec-conformance check
passed - the bug was *in the spec*, latent and unreachable until a sibling feature
gets wired. Lesson: add a "shared/persistent state bleed" lens when reviewing
additions to long-lived singletons.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260615-0913-delayed-completion-coroutine-needs-singleshot-latch|A delayed-completion coroutine that still reads input double-fires without a single-shot latch]] - wspolne: state-management, input, minigame
- [[20260724-1817-diegetic-button-overlap-steal|Powiekszone collidery klikania ciasnych guzikow 3D + wybor "pierwsze trafione pudelko" = sasiad kradnie klik]] - wspolne: input, minigame
<!-- /POWIAZANE:auto -->
