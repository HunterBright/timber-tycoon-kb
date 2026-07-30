---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [unity, physics, raycast, backface, water, meshcollider, layers]
severity: medium
time_lost: "~30 min"
date: 2026-07-22
status: draft
applies_to: [unity-6]
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
- [[unstuck-nearest-valid-ground-ring-search]]
