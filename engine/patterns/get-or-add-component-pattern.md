---
title: GetOrAddComponent Extension Method
type: pattern
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- components
- extension-method
- refactoring
applies_to: []
source: ''
suggested-category: engine/patterns
---

# GetOrAddComponent Extension Method

## When to use
Any place you would write `AddComponent<T>()` in a setup script, spawn handler, or editor tool - especially if that code might run more than once (re-runs, respawns, parking respawn).

## Steps
Define once in `Assets/Project/Scripts/Extensions/GameObjectExtensions.cs`:
```csharp
public static T GetOrAddComponent<T>(this GameObject go) where T : Component {
    T comp = go.GetComponent<T>();
    return comp ?? go.AddComponent<T>();
}
```

Use everywhere instead of bare `AddComponent`:
```csharp
vehicle.GetOrAddComponent<NPCVehicle>();
```

## Why this works
Calling `AddComponent<T>()` twice creates two instances of the same component. Both run Awake/Start. The second one wins at runtime; the first one is an orphaned instance that wastes memory and can cause subtle bugs. `GetOrAddComponent` makes the operation idempotent.

## Trade-offs
Tiny overhead: one extra `GetComponent` call. Always worth it for any setup code that might run multiple times.

## Variants
Same principle as `GetOrCreateChild(name)`, `GetOrAddLayer(tag)` - any "ensure X exists, create if missing" pattern.

<!-- WERYFIKATOR 2026-08-05 -->
Weryfikacja 2026-08-05: sam pomysl (jedno wywolanie zamiast slepego dokladania komponentu)
nie budzi zastrzezen, ale przyklad kodu uzywa `??` na obiekcie Unity, czego zabrania
nasz wlasny wpis [[20260702-1610-fake-null-so-null-conditional-trap]] (status verified):
"W polach Unity nigdy `?.` ani `??` - zawsze jawne `if (x != null)`". Roznica jest realna
tylko dla komponentu **zniszczonego, a jeszcze nie sprzatnietego** - wtedy `??` przepuszcza
martwa skorupe zamiast utworzyc nowy komponent. Zastrzezenie: oficjalna regula analizatora
Unity (UNT0008) opisuje wprost tylko operator `?.`, nie `??`, wiec sprzecznosc jest
z nasza wlasna reguła, a nie z litera dokumentacji producenta.
Zrodlo: https://github.com/microsoft/Microsoft.Unity.Analyzers/blob/main/doc/UNT0008.md
