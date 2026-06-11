---
type: anti-pattern
project: timber-tycoon
suggested-category: engine/anti-patterns
tags: [unity, editor-tooling, waypoints, codegen, npc-parking]
severity: medium
date: 2026-05-22
status: draft
---

# ANTI-PATTERN: Generator Destroys Both Paths With No Guard

## The Trap
`GenerateParkingApproachWaypoints.cs` unconditionally destroys **and** rebuilds **both** the `ReversePath` and the `ExitPath` child objects on every run — with no per-path guard and no awareness of a separately-authored `BranchExitPath`.

## Why It Bites
When you re-run the generator to tune **one** path (e.g. the reverse curve), it silently regenerates the **other** path too, overwriting any hand-tuning and reconnecting endpoints to formula-derived coordinates. In Timber Tycoon this caused a regression that looked like a parking alignment bug but was actually the exit path being clobbered — the hand-placed `BranchExitPath` connection was replaced with a formula coordinate that pointed nowhere useful.

Debugging is hard because the symptom is in the wrong subsystem: the path you edited looks fine, but the path you never touched is broken.

## Correct Approach
Split generation per-path — either separate menu items or a guard flag:

```csharp
// Option A: separate menu items
[MenuItem("Timber Tycoon/Generate ReversePath Only")]
static void GenerateReversePath() { ... }

[MenuItem("Timber Tycoon/Generate ExitPath Only")]
static void GenerateExitPath() { ... }

// Option B: guard flag in the shared generator
public bool regenerateReversePath = true;
public bool regenerateExitPath = true;
```

Also make the generator `BranchExitPath`-aware: instead of computing exit endpoints from a formula, read the actual position of the hand-placed `BranchExitPath` start waypoint and connect to it.

## Detection Pattern
If editing/tuning one generated path breaks a different path you never touched — check whether the generator rebuilds everything unconditionally on every run.

Confirm with a log: add `Debug.Log($"[Generator] Destroying {child.name}")` inside the destroy loop and watch what fires when you run it.

## See also
[[custom-editor-pattern-for-generators]]
