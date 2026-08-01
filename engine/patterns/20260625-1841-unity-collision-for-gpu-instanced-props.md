---
title: Dodawanie kolizji do propów rysowanych GPU instancingiem
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-06-25'
project: Kerf - Sawmill Tycoon
tags:
- unity
- gpu-instancing
- physics
- meshcollider
- RenderMeshInstanced
- performance
- decor
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Dodawanie kolizji do propów rysowanych GPU instancingiem

## Problem / kontekst
Dekoracja masowa (kamienie, krzaki, trawa) bywa rysowana przez `Graphics.RenderMeshInstanced`
/ `Graphics.DrawMeshInstanced` z danych w blobie (macierze TRS per instancja) - **bez per-instancja
GameObjectów**, dla wydajności. Taki prop to tylko „obrazek": nie ma Transformu, nie ma collidera,
więc gracz/pojazd przechodzą przez niego na wylot. Nie da się „dopiąć collidera", bo nie ma do czego.

## Wzorzec rozwiązania (runtime collider-proxy)
Dorobić warstwę fizyki OBOK warstwy renderującej, nie ruszając renderowania:
1. Companion `MonoBehaviour` na tym samym obiekcie co renderer. Na `Start()` **re-parsuje ten sam blob**
   instancji (ta sama kolejność grup = ta sama lista mesh/material).
2. Filtruje, które GRUPY mają dostać kolizję (np. po prefiksie etykiety - średnie kamienie/głazy TAK,
   drobne kamyki NIE), żeby nie tworzyć tysięcy zbędnych brył.
3. Dla każdej instancji tworzy dziecko collider-only (`MeshCollider`, BEZ MeshRenderer/MeshFilter),
   ustawia world TRS = (pos, rot, scale) z blobu, `convex = false` (statyczne, dokładne),
   warstwa kolizji = Default.
4. Po pętli `Physics.SyncTransforms()` (konieczne, gdy `m_AutoSyncTransforms = 0`).

## Pułapki (oszczędzają godziny)
- **MeshCollider w runtime wymaga `Read/Write Enabled` (isReadable) na siatce.** FBX importują się
  domyślnie z `isReadable: 0`; runtime `MeshCollider.sharedMesh` wtedy CICHO nie zadziała. Włącz
  Read/Write na tych kilku siatkach (koszt RAM znikomy dla low-poly). Dodaj strażnik `if (!mesh.isReadable)`
  → log ostrzeżenia zamiast cichego braku kolizji. (Jeśli generujesz collidery w EDYTORZE, a nie w runtime -
  siatka jest czytelna i ten wymóg odpada.)
- **Concave MeshCollider jest OK i dokładny dla obiektów STATYCZNYCH** (nie ruszają się, brak Rigidbody).
  `convex = true` jest potrzebny tylko dla ruszających się/Rigidbody colliderów.
- **Zero bloatu sceny:** rodzic proxy z `HideFlags.DontSave` + budowa w runtime → collidery NIGDY się nie
  serializują do sceny i nie zostają „osierocone" po ponownym rozsianiu (re-scatter). Save sceny czysty.
- Setki statycznych mesh colliderów to dla PhysX drobiazg (broadphase raz, ~0 kosztu/klatkę gdy nic się
  o nie nie ociera). CharacterController i Rigidbody blokuje każdy nie-trigger collider, jeśli macierz
  warstw na to pozwala (Default zwykle koliduje ze wszystkim).

## Alternatywy (rozważone, odrzucone tu)
- Zamiana instancingu na realne prefab-GameObjecty → traci wydajność i bloatuje scenę.
- Jeden scalony collision-mesh per grupa → mniej komponentów, ale wymaga re-bake przy zmianie scatteru.
- Prymitywy (box/sphere) per prop → tanio, ale mniej dokładne niż mesh; użyj gdy kształt nieistotny.

## Case study (Timber Tycoon)
`InstancedDecorRenderer` rysował 1137 kamieni (Pebble/Rock/Boulder) z blobu - zero kolizji. Dodano
`InstancedRockColliders` (runtime proxy) dla 363 średnich kamieni+głazów; kamyki celowo do przejścia.
Wymagało `isReadable: 0→1` na 5 FBX. Wynik: gracz i auto blokują się o kamienie, wizual i FPS bez zmian.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260623-1508-instanced-grass-cards|Performant stylized grass: textured cards + GPU instancing (no GameObjects)]] - wspolne: gpu-instancing, performance
- [[20260710-2115-collider-from-first-meshfilter-antipattern|Collider z GetComponentInChildren&lt;MeshFilter&gt; na wielosiatkowym FBX = collider z fragmentu modelu]] - wspolne: meshcollider, physics
- [[20260628-1615-footstep-surface-raycast-from-feet|Detekcja nawierzchni pod graczem: strzelaj raycastem OD STÓP, nie od środka postaci]] - wspolne: meshcollider, physics
- [[20260628-1702-editmode-collider-cook-scatter-mask|Edit-mode placement that excludes geometry via physics fails for non-convex runtime/embedded MeshColliders]] - wspolne: meshcollider, physics
- [[20260713-1420-convex-meshcollider-swallows-hollow-interiors|Convex MeshCollider na skorupie połyka jej wnętrze - promienie nigdy nie trafią w części w środku]] - wspolne: meshcollider, physics
- [[20260713-1425-runtime-meshcollider-needs-readable-mesh-in-builds|MeshCollider tworzony w runtime działa w Edytorze i cicho pada w buildzie (isReadable)]] - wspolne: meshcollider, physics
<!-- /POWIAZANE:auto -->
