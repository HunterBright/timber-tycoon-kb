---
title: Self-Collision Compound BoxColliders → Physics.IgnoreCollision
type: lesson
status: needs-reproduction
confidence: low
verified: '2026-07-30'
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- physics
- colliders
- compound
- rigidbody
- vehicle
applies_to:
- unity-projects
source: zweryfikowane reprodukcja i dokumentacja 2026-07-30, patrz AUDYT-SPORNYCH-WPISOW
severity: high
time_lost: ~40 min
suggested-category: engine/lessons
audit_verdict: BLEDNY
---

# Self-Collision Compound BoxColliders → Physics.IgnoreCollision

> [!warning] Ten wpis zostal zweryfikowany 2026-07-30 i werdykt brzmi: **BLEDNY**
>
> Przyczyna opisana w tym wpisie zostala **obalona reprodukcja** w Unity 6000.5.1f1: zderzaki nalezace do jednego ciala fizycznego NIE odpychaja sie wzajemnie (ruch 0,00000 m wobec 6,72825 m w ukladzie kontrolnym).
> Zalecana petla `Physics.IgnoreCollision` po parach dzieci najprawdopodobniej nic nie robi.
> Obejscie dziala, ale dla INNEJ przyczyny: zagniezdzonego Rigidbody.
>
> Pelne uzasadnienie, dowody i proponowane poprawki: [[AUDYT-SPORNYCH-WPISOW]].
> Tresc ponizej NIE zostala jeszcze przepisana - czytaj ja z ta uwaga.


## Problem
Vehicles with compound BoxColliders (Cabin + Flatbed + body parts) explode or launch into the air on spawn. The Rigidbody violently reacts to internal forces that weren't there in the editor.

## Root cause
Compound colliders on the same Rigidbody are NOT automatically excluded from colliding with each other. Unity's physics engine detects overlap between Cabin BoxCollider and body BoxCollider, applies separation forces - resulting in the vehicle launching or jittering violently at spawn.

## Solution
At Awake, iterate all child `Collider` components and call `Physics.IgnoreCollision` for every pair:

```csharp
void Awake() {
    var cols = GetComponentsInChildren<Collider>();
    for (int i = 0; i < cols.Length; i++)
        for (int j = i + 1; j < cols.Length; j++)
            Physics.IgnoreCollision(cols[i], cols[j], true);
}
```

Caveat: only applies to colliders on the SAME Rigidbody. Child Rigidbodies need separate handling.

## What didn't work
Assuming Unity auto-excludes intra-object collisions for compound setups.

## Transferability
Any Unity project using compound collider hierarchies on a single Rigidbody - robots, vehicles, modular characters, destructible objects - needs this fix. Standard boilerplate for any Awake that sets up physics-driven objects with multiple collider children.

## Related
- [Forward axis quirk](forward-axis-blender-fbx-quirk.md)
- [Vehicle interaction zones as triggers](../patterns/vehicle-interaction-zones-as-triggers.md)
