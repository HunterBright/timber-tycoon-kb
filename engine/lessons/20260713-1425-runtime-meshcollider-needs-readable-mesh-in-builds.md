---
title: MeshCollider tworzony w runtime działa w Edytorze i cicho pada w buildzie (isReadable)
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-13'
project: Kerf - Sawmill Tycoon
tags:
- unity
- meshcollider
- build
- isReadable
- works-on-my-machine
- physics
applies_to:
- unity
source: 'https://docs.unity3d.com/6000.3/Documentation/ScriptReference/Mesh-isReadable.html (weryfikacja 2026-08-01: dokumentacja potwierdza mechanizm - przy isReadable=false Unity usuwa siatke z pamieci procesora, a odczyt jej danych rzuca blad - ale NIE opisuje wprost przypadku collidera dokladanego w runtime; do odtworzenia u nas)'
severity: high
time_lost: 0 (złapane w recenzji przed buildem)
promoted: '2026-07-30'
---

# MeshCollider tworzony w runtime działa w Edytorze i cicho pada w buildzie (isReadable)

## Problem

Kod dodaje collider do modelu w czasie gry:

```csharp
var mc = go.AddComponent<MeshCollider>();
mc.sharedMesh = meshFilter.sharedMesh;
```

W Edytorze działa **zawsze**. W zbudowanej grze - jeśli model ma w importerze `Read/Write Enabled = false` (domyślnie WYŁĄCZONE) - collider powstaje **pusty**: promienie w nic nie trafiają, gracz przechodzi przez obiekt, klikalne rzeczy przestają być klikalne. W logu leci `CollisionMeshData couldn't be created because the mesh has been marked as non-accessible`, ale nikt tego nie widzi, bo build zwykle nie jest oglądany z konsolą.

## Root cause

- W Edytorze siatka ma zawsze kopię po stronie CPU (trzyma ją AssetDatabase), więc cooking collidera się udaje niezależnie od `isReadable`.
- W buildzie siatka z `isReadable: 0` po wgraniu na GPU jest usuwana z pamięci CPU. Nie ma z czego zbudować danych kolizji.
- Dlaczego mesh collidery na geometrii poziomu działają mimo `isReadable: 0`? Bo są **zserializowane** w scenie/prefabie - pipeline buildu widzi referencję i zapieka dane kolizji z góry. `AddComponent` w runtime tej ścieżki nie ma.

To jest podręcznikowy „works on my machine": bug nie istnieje w jedynym środowisku, w którym się testuje.

## Solution

Do wyboru, zależnie od sytuacji:

1. **`isReadable: 1`** w importerze modelu (`.meta`). Koszt: kopia siatki w RAM. Dla małych modeli (setki wierzchołków) pomijalny. Edycja `.meta` nie rusza GUID-ów.
2. **`addColliders: 1`** (Generate Colliders w importerze) - collidery są w assecie, więc zserializowane; działają w buildzie bez `isReadable` i bez cookingu w runtime. Minus: importer nadaje collider KAŻDEJ siatce modelu, także tam, gdzie go nie chcesz.
3. Nie dodawać colliderów w runtime - trzymać je w prefabie.

## What didn't work

Nic - to złapała recenzja adwersarialna przed buildem. Ale warto odnotować **pułapkę rozumowania**: „przecież to samo już działa u nas w grze (drzewa robią tak od miesięcy)" nie jest dowodem, że działa w buildzie. Jeśli produkt nie był jeszcze budowany, **cała ta klasa bugów śpi**.

## Transferability

Uniwersalne dla Unity. Warto przy pierwszym buildzie każdego projektu zrobić grep:

```
AddComponent<MeshCollider>
```

i dla każdego trafienia sprawdzić `isReadable` w `.meta` modelu, z którego bierze się siatka. To samo dotyczy każdego runtime'owego czytania siatki: `mesh.vertices`, `mesh.triangles`, `Mesh.CombineMeshes`, bake'owanie nawigacji z siatek.

Szersza reguła: **wszystko, co w Edytorze bierze dane z assetu „bo są pod ręką", w buildzie może ich nie zastać.** Pierwszy build to nie formalność - to osobny test.

## Related
- [[20260713-1420-convex-meshcollider-swallows-hollow-interiors]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260713-1830-runtime-meshcollider-needs-readwrite-and-editor-cannot-prove-it|Runtime MeshCollider wymaga Read/Write Enabled - a Edytor NIE JEST w stanie tego udowodnić]] - wspolne: isreadable, build, meshcollider
- [[20260628-1615-footstep-surface-raycast-from-feet|Detekcja nawierzchni pod graczem: strzelaj raycastem OD STÓP, nie od środka postaci]] - wspolne: meshcollider, physics
- [[20260628-1702-editmode-collider-cook-scatter-mask|Edit-mode placement that excludes geometry via physics fails for non-convex runtime/embedded MeshColliders]] - wspolne: meshcollider, physics
- [[20260713-1420-convex-meshcollider-swallows-hollow-interiors|Convex MeshCollider na skorupie połyka jej wnętrze - promienie nigdy nie trafią w części w środku]] - wspolne: meshcollider, physics
- [[20260722-2055-raycast-w-gore-nie-widzi-tafli-wody|"Czy jestem pod wodą?" - promień w GÓRĘ nic nie zobaczy (backface)]] - wspolne: meshcollider, physics
- [[20260728-1110-meshcollider-niewypukly-z-rigidbody-gubi-kolizje|Zderzak z pelnej siatki na obiekcie z fizyka = obiekt znika ze swiata]] - wspolne: meshcollider, physics
<!-- /POWIAZANE:auto -->
