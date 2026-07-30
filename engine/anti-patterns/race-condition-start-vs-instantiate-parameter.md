---
title: 'ANTI-PATTERN: Start() Reads SO Before Parent Sets It'
type: anti-pattern
status: needs-reproduction
confidence: low
verified: '2026-07-30'
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- lifecycle
- race-condition
- instantiate
- scriptableobject
applies_to: []
source: zweryfikowane reprodukcja i dokumentacja 2026-07-30, patrz AUDYT-SPORNYCH-WPISOW
suggested-category: engine/anti-patterns
audit_verdict: CZESCIOWO BLEDNY
---

# ANTI-PATTERN: Start() Reads SO Before Parent Sets It

> [!warning] Ten wpis zostal zweryfikowany 2026-07-30 i werdykt brzmi: **CZESCIOWO BLEDNY**
>
> Zly winowajca: to nie `Start()` czyta dane za wczesnie, tylko `Awake()` i `OnEnable()`.
> Blok oznaczony we wpisie jako "bledny kod" jest w rzeczywistosci poprawny.
> Wszystkie trzy sposoby naprawy zostaja - dzialaja.
>
> Pelne uzasadnienie, dowody i proponowane poprawki: [[AUDYT-SPORNYCH-WPISOW]].
> Tresc ponizej NIE zostala jeszcze przepisana - czytaj ja z ta uwaga.


## The trap
Instantiate a prefab, then assign a ScriptableObject reference to it right after. Seems fine - the assignment happens right there in the code. But `Start()` already ran with `null`.

```csharp
// BUGGY
GameObject tree = Instantiate(prefab);
GrowingTree gt = tree.GetComponent<GrowingTree>();
gt.treeTypeData = data; // TOO LATE - Start() already ran with null
```

## Why it fails
`Instantiate` triggers `Start()` immediately (in the same frame, before the calling code continues). If the component's `Start()` reads `treeTypeData` and it's null, the component initializes with defaults or throws `NullReferenceException`. The assignment on the next line arrives too late.

## Symptoms
- `NullReferenceException` in a `Start()` method for a freshly instantiated object
- Default/empty values used instead of injected SO data
- Only fails when spawned via code; works fine when prefab is in scene (Inspector-assigned SO)

## Correct approach
**Fix 1 (preferred):** Explicit initialization method called after assignment:
```csharp
GameObject tree = Instantiate(prefab);
GrowingTree gt = tree.GetComponent<GrowingTree>();
gt.treeTypeData = data;
gt.InitializeWithSO(); // explicit init after parent has set fields
```

**Fix 2:** Move SO-dependent setup out of `Start()`, into `Awake()` for SO-independent parts, and use a coroutine or first `Update()` for SO-reading parts.

**Fix 3 (cleanest):** Factory method:
```csharp
GrowingTree gt = TreeFactory.Spawn(prefab, data); // sets SO before activating
```

Rule: child lifecycle methods must not assume parent-injected fields are set.

See also: [[awake-init-for-isaveable-with-dependencies]] for the ISaveable variant of this lifecycle ordering problem.
