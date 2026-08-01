---
title: '"Czy jestem pod wodą?" - promień w GÓRĘ nic nie zobaczy (backface)'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-22'
project: Kerf - Sawmill Tycoon
tags:
- unity
- physics
- raycast
- backface
- water
- meshcollider
- layers
applies_to:
- unity-6
source: ''
severity: medium
time_lost: ~30 min
promoted: '2026-07-30'
---

# "Czy jestem pod wodą?" - promień w GÓRĘ nic nie zobaczy (backface)

## Problem
Trzeba było odpowiedzieć na pytanie "czy nad tym punktem gruntu stoi woda", żeby odróżnić
brzeg jeziora od jego dna. Naturalny odruch: puścić promień z punktu gruntu W GÓRĘ i
sprawdzić, czy trafi w taflę. Taki test **zawsze zwraca 'sucho'**, także po pas w wodzie.

## Root cause
Tafla wody to płaska siatka z normalnymi skierowanymi do góry. Promień lecący od dołu
trafia w jej TYLNĄ stronę, a `Physics.queriesHitBackfaces` jest domyślnie `false` -
silnik takiego trafienia nie zgłasza. Dotyczy każdej jednostronnej geometrii, nie tylko wody:
sufity jaskiń, kurtyny, siatki terenu bez ścian bocznych.

## Solution
Mierz **z góry w dół** i porównuj wysokości: promień z pułapu (wysoko nad terenem) w dół,
trafienie w taflę porównaj z wysokością gruntu. Woda nad gruntem = punkt jest pod wodą.

Drugi wniosek: **nie szukaj wody po numerze warstwy.** Skrypty setupowe ustawiają warstwę
raz, a scena żyje własnym życiem i po roku nie musi się z nimi zgadzać. Pewniejsze jest
zebranie colliderów po KOMPONENCIE gameplayowym (u nas `WaterZone`) i odpytywanie ich
bezpośrednio przez `Collider.Raycast` - to omija maski warstw i macierz kolizji.

`Collider.Raycast` na `MeshCollider` **nie wymaga** Read/Write Enabled na modelu (działa na
ugotowanych danych kolizji), więc ten sposób nie wpada w klasyczną pułapkę "Edytor kłamie".

## What didn't work
- Promień w górę na maskę warstwy Water - zawsze pusto (backface).
- Poleganie na macierzy kolizji warstw jako źródle prawdy o tym, czy woda jest przeszkodą:
  macierz mówiła "Water koliduje ze wszystkim", a gracz i tak brodził w rzece. Macierz
  opisuje warstwy, nie to, czy collider jest triggerem.

## Transferability
Każda gra z wodą, mgłą, sufitem jaskini albo jednostronną kurtyną, gdzie kod pyta
"co jest nade mną". Wzorzec "mierz z góry w dół i porównaj wysokości" jest uniwersalny.

## Related
- [[20260722-2050-unstuck-nearest-valid-ground-ring-search]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260628-1615-footstep-surface-raycast-from-feet|Detekcja nawierzchni pod graczem: strzelaj raycastem OD STÓP, nie od środka postaci]] - wspolne: meshcollider, raycast, physics
- [[20260628-1702-editmode-collider-cook-scatter-mask|Edit-mode placement that excludes geometry via physics fails for non-convex runtime/embedded MeshColliders]] - wspolne: meshcollider, raycast, physics
- [[20260713-1420-convex-meshcollider-swallows-hollow-interiors|Convex MeshCollider na skorupie połyka jej wnętrze - promienie nigdy nie trafią w części w środku]] - wspolne: meshcollider, raycast, physics
- [[20260723-1746-ignorecollision-wiped-on-collider-disable|Physics.IgnoreCollision znika przy wyłączeniu collidera - dla przełączanych colliderów używaj par warstw]] - wspolne: layers, physics
- [[20260628-1140-conform-road-mesh-to-edited-terrain|Conforming an existing road/decal mesh to terrain that was edited later]] - wspolne: meshcollider, raycast
- [[20260713-1425-runtime-meshcollider-needs-readable-mesh-in-builds|MeshCollider tworzony w runtime działa w Edytorze i cicho pada w buildzie (isReadable)]] - wspolne: meshcollider, physics
<!-- /POWIAZANE:auto -->
