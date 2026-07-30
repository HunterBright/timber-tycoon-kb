---
title: A service locator's Get() must distinguish OPTIONAL from REQUIRED lookups, or it false-alarms at start/teardown
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-15'
project: Kerf - Sawmill Tycoon
tags:
- unity
- service-locator
- singleton
- lifecycle
- logging
- initialization-order
- false-positive
applies_to: []
source: ''
suggested-category: engine/lessons
---

# A service locator's Get() must distinguish OPTIONAL from REQUIRED lookups, or it false-alarms at start/teardown

## Severity
Low (noise, not a crash) — but the noise is loud: dozens of red `LogError` lines on
every Play start AND stop, which buries real errors and erodes trust in the console.

## Context
A typical service locator: `static T Get<T>()` that, on a miss, `Debug.LogError(...)`
and returns null. Consumers register themselves with managers in lifecycle hooks:

```csharp
void OnEnable()  { var sm = Services.Get<SaveManager>(); if (sm != null) sm.Register(this); }
void OnDisable() { var sm = Services.Get<SaveManager>(); if (sm != null) sm.Unregister(this); }
```

## The trap
Unity's `Awake`/`OnEnable` order across objects is nondeterministic, and on Play-stop /
scene unload objects are destroyed in arbitrary order. So:
- At START, a consumer's `OnEnable` can run before the manager's `Awake` registered it.
- At TEARDOWN, the manager can be destroyed (and self-unregister) before consumers'
  `OnDisable`/`OnDestroy` run.

In both cases the consumer's `Get<T>()` misses and fires a red `LogError`, even though
the consumer ALREADY null-checks the result and behaves correctly. The error is a false
alarm: the caller treats absence as acceptable, but `Get` treats it as a bug. With dozens
of `ISaveable`/registry/event-subscriber objects, this is dozens of red lines per session.

## Fix
Give the locator TWO accessors and make the call site declare intent:
```csharp
public static T  Get<T>()                      // REQUIRED — logs error on miss (real bug)
public static bool TryGet<T>(out T service)     // OPTIONAL — silent, returns false on miss
public static bool Has<T>()                      // existence check, silent
```
Rule of thumb: **if the call site null-checks the result, it is an optional lookup → use
TryGet (silent). If it does not null-check (it assumes the service must exist), keep Get
(loud).** Migrate every null-checked lifecycle lookup:
```csharp
if (Services.TryGet<SaveManager>(out var sm)) sm.Register(this);
```
This is behavior-preserving (same null/non-null) and removes the false alarms while
keeping the diagnostic for genuinely-missing required services.

## Note — silencing vs. real ordering bugs
Silencing the lookup does NOT fix initialization order; it makes "absent here is fine"
explicit. If an object MUST be registered regardless of order (e.g. for save), silencing
can hide a real "never registered → never saved" bug. So pair the migration with a quick
check that critical consumers actually register (or enforce order via Script Execution
Order / a bootstrap that registers core managers first).

## Why transferable
Any Unity project using a ServiceLocator/registry pattern with MonoBehaviours that
self-register in lifecycle hooks hits this. The optional-vs-required distinction (TryGet
vs Get) is a general API-design lesson, not project-specific.
