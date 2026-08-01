---
title: Runtime MeshCollider wymaga Read/Write Enabled - a Edytor NIE JEST w stanie tego udowodnić
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-13'
project: Kerf - Sawmill Tycoon
tags:
- unity
- build
- meshcollider
- isreadable
- read-write-enabled
- physx
- editor-vs-build
- verification
applies_to:
- unity
source: ''
severity: high
time_lost: ~6h (audyt + 2 buildy + diagnoza fałszywego planu awaryjnego)
promoted: '2026-07-30'
---

# Runtime MeshCollider wymaga Read/Write Enabled - a Edytor NIE JEST w stanie tego udowodnić

## Problem

Kod, który w runtime robi `AddComponent<MeshCollider>()` + `mc.sharedMesh = mf.sharedMesh` na siatce
z importowanego FBX, **działa w Edytorze zawsze**, ale w buildzie collider powstaje **PUSTY**
(`mc.bounds == 0,0,0`, raycast go nie widzi). Silnik loguje:

```
CollisionMeshData couldn't be created because the mesh has been marked as non-accessible. Mesh name "X"
```

Objawy gameplay: gracz **przechodzi na wylot** przez obiekty, które miały mieć kolizję; obiekty
mające być klikalne stają się nietrafialne (u nas: softlock minigry, bo kliknięcie było warunkiem
postępu). W Edytorze wszystko działa idealnie, więc bug jest **niewidoczny do pierwszego buildu**.

## Root cause

`isReadable: 0` (Read/Write Enabled OFF w ModelImporterze) to ustawienie **domyślne** Unity.

- W **Edytorze** AssetDatabase trzyma kopię siatki po stronie procesora **niezależnie od flagi** →
  PhysX zawsze ma z czego ugotować collider.
- W **buildzie** siatka z `isReadable: 0` jest po wysłaniu na GPU **kasowana z pamięci procesora** →
  runtime cook nie ma danych.

Kluczowa asymetria, która myli: collidery **zapisane w scenie/prefabie działają mimo `isReadable: 0`**,
bo pipeline buildu zapieka im `CollisionMeshData` z góry. **`AddComponent` w runtime tej ścieżki nie ma.**
Dlatego „przecież collidery działają" nie jest dowodem - trzeba sprawdzić, KTÓRE powstają w runtime.

## Solution

1. **Audytuj po grafie danych, nie po nazwach plików.** Znajdź każdy runtime'owy `AddComponent<MeshCollider>`
   / przypisanie `sharedMesh` i prześledź, z jakich assetów pochodzą siatki (SO → prefab → MeshFilter → mesh
   → `AssetDatabase.GetAssetPath(mesh)`).
2. **Włącz `isReadable` TYLKO tam.** Każdy niepotrzebnie czytelny model to dublet siatki w RAM.
   Rób to **przez API importera** (`ModelImporter.isReadable = true; SaveAndReimport()`), nie edycją
   tekstu `.meta` - GUID-y zostają nietknięte, a `Library` przebudowuje się spójnie.
3. **Dodaj głośny guard w kodzie** - cicha awaria musi stać się widoczna:
   ```csharp
   if (!mf.sharedMesh.isReadable) { Debug.LogError("... brak Read/Write -> w BUILDZIE brak kolizji"); continue; }
   ```
   Tam, gdzie brak collidera oznacza softlock, degraduj z gracją (np. `BoxCollider` z `mesh.bounds` -
   `bounds` to zserializowane metadane i **działają też dla nieczytelnej siatki**).
4. **Test-strażnik chodzący po grafie danych** (nie po zahardkodowanej liście plików) - wtedy nowy asset
   dodany za pół roku zapali czerwone światło sam.

## What didn't work

**`Mesh.UploadMeshData(true)` NIE symuluje buildu w Edytorze.** To był mój plan awaryjny („udowodnimy
mechanizm bez buildu") i jest **fałszywy**. Zmierzone (Unity 6000.5.1f1):

```
kopia siatki po UploadMeshData(true): isReadable = False
  .vertices.Length  = 70    <-- dane NADAL SĄ
  .triangles.Length = 78
  MeshCollider(kopia).Raycast = True   <-- collider ugotował się normalnie
```

Edytor trzyma dane siatki **mimo** `isReadable == false` i **mimo** jawnego żądania ich zwolnienia.
Wniosek: **nie ma drogi na skróty. Jedynym dowodem na tę klasę błędów jest prawdziwy build.**

Inne odrzucone drogi:
- `addColliders: 1` w importerze - kusi (collider zapieczony w assecie, zero kosztu RAM), ale:
  (a) daje collider **każdej** siatce modelu, (b) tylko **non-convex**, (c) **nie zadziała**, gdy prefab
  jest „płaski" (ręcznie zbudowana hierarchia wskazująca tylko na siatki z FBX, bez instancji prefabu
  modelu) - collider z importera wyląduje na obiektach, których taki prefab nigdy nie tworzy.
  U nas naprawiłby 22 z 26 modeli i zostawił 4, do których nikt by już nie zajrzał. **Cichy fix częściowy
  jest gorszy niż brak fixa.**
- Sprawdzanie samej flagi `isReadable` jako kryterium PASS - nic nie dowodzi: collider z prefabu działa
  przy `false`, a ustawiona flaga nie gwarantuje udanego cooka.

## Wzorzec weryfikacji, który zadziałał

**Sonda uruchamiana w prawdziwym buildzie, bramkowana argumentem wiersza poleceń:**

```csharp
[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.AfterSceneLoad)]
static void Bootstrap()
{
    // NIE bramkuj na Application.isEditor - sonda ma dzialac wlasnie w buildzie
    if (!HasArg("-meshaudit")) return;
    ...
}
```

Trzy zasady, bez których sonda jest bezwartościowa:

1. **Wykonuje DOKŁADNIE kod gry**, nie jego kopię. Wyciągnij tworzenie colliderów do publicznego statyku
   i wołaj go z gry ORAZ z sondy - inaczej sonda i gra rozjadą się przy pierwszym refaktorze.
2. **Weryfikuje BEHAWIORALNIE**: `Collider.Raycast` w bryłę z kilku kierunków (pyta wprost o KSZTAŁT
   w PhysX, bez warstw i sceny fizyki). Pusty collider zwraca `false` ze wszystkich.
3. **Detektor musi być zwalidowany** na znanym-złym przypadku (pusta `new Mesh()` → collider bez kształtu →
   detektor MUSI dać `false`) i znanym-dobrym. Bez tego jego werdykt nic nie znaczy.

Wynik do pliku obok `.exe` + `Application.Quit(failed == 0 ? 0 : 1)` → maszynowo sprawdzalne, bez grania.
Efekt: **PRZED 7/35 PASS + 582 błędy silnika; PO 35/35 PASS, zero błędów.**

## Transferability

Dotyczy **każdego projektu Unity**, niezależnie od gatunku. `isReadable: 0` to domyślne ustawienie
importera, a runtime'owy `AddComponent<MeshCollider>` to bardzo częsty wzorzec (proceduralne ustawianie
obiektów, spawn z prefabów, obiekty składane w kodzie).

Ta sama pułapka dotyczy **każdego runtime'owego odczytu danych siatki**: `mesh.vertices`, `mesh.triangles`,
`mesh.uv`, `mesh.colors`, `Instantiate(mesh)`, a także **`RaycastHit.triangleIndex` i
`RaycastHit.textureCoord`** (te dwa też wymagają czytelnej siatki trafionego collidera!).

**Siatki generowane w kodzie (`new Mesh()`) i osadzone w scenie są czytelne z definicji** - ich nie trzeba
ruszać. Odróżnienie „siatka z FBX" od „siatka z kodu" to pierwszy krok każdego takiego audytu.

Najszersza lekcja: **projekt, który nigdy nie był budowany, hoduje całą klasę uśpionych błędów.**
Buduj wcześnie, choćby raz.

## Related
- [[20260713-1845-monobehaviour-class-must-match-filename]]
- [[20260713-1900-build-early-never-built-project-hides-editor-only-bugs]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260715-1150-meshcollider-nonreadable-native-crash|MeshCollider na siatce bez Read/Write = TWARDY natywny crash builda przy odsloniecie (nie w Edytorze)]] - wspolne: read-write-enabled, physx, meshcollider
- [[20260713-1425-runtime-meshcollider-needs-readable-mesh-in-builds|MeshCollider tworzony w runtime działa w Edytorze i cicho pada w buildzie (isReadable)]] - wspolne: isreadable, build, meshcollider
- [[20260719-1210-unity-build-freshness-check-dll-not-exe|Świeżość builda Unity sprawdzaj po DLL z kodem gry, nie po .exe]] - wspolne: verification, build
- [[20260713-1900-build-early-never-built-project-hides-editor-only-bugs|Projekt, który nigdy nie był budowany, hoduje całą klasę uśpionych błędów]] - wspolne: verification, build
- [[20260714-2245-unity-batchmode-returns-before-build-finishes|Unity w trybie wsadowym WRACA, zanim build się skończy - i sonda daje fałszywe zielone światło]] - wspolne: verification, build
<!-- /POWIAZANE:auto -->
