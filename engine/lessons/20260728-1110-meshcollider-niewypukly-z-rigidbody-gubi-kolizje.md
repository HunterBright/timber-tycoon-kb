---
title: Zderzak z pelnej siatki na obiekcie z fizyka = obiekt znika ze swiata
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- physics
- meshcollider
- rigidbody
- convex
- pickup
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Zderzak z pelnej siatki na obiekcie z fizyka = obiekt znika ze swiata

## Objaw

Obiekt, ktory ma zostac podniesiony albo wyrzucony (pniak wykopany z ziemi,
przedmiot do noszenia), po wlaczeniu fizyki przelatuje przez teren i przepada.
Wyglada jak "zniknal". W Edytorze na scenie wszystko wyglada poprawnie.

## Przyczyna

`MeshCollider` z `convex = false` **nie moze** nalezec do obiektu z niekinematycznym
`Rigidbody`. Unity nie potrafi policzyc dla takiego ksztaltu masy ani kontaktow,
wiec kolizja przestaje dzialac - obiekt spada przez wszystko.

Pulapka polega na tym, ze **w chwili budowania prefabu wszystko jest w porzadku**:
zderzak z pelnej siatki jest tam poprawny i tani, bo obiekt stoi nieruchomo.
Rigidbody dochodzi DOPIERO w trakcie gry (u nas: `PopStumpOut` przy wykopywaniu).
Blad rodzi sie wiec w innym miejscu, niz wybucha.

## Regula

Zanim dasz obiektowi `MeshCollider`, zapytaj: **czy ten obiekt kiedykolwiek dostanie
fizyke?** Jesli tak (podnoszenie, rzucanie, wypadanie, wybuch):

- `convex = true` na MeshColliderze (limit 255 wielokatow, wystarcza dla pniaka
  czy skrzynki), albo
- prosty zderzak: Box / Capsule / Sphere.

Zderzak z pelnej siatki zostaw wylacznie obiektom, ktore stoja: budynki, teren,
maszyny, drzewa rosnace.

## Drugi trop z tego samego dnia

Przy okazji wyszlo, ze obiekt mial DWA zderzaki naraz (stara kapsula z poprzedniej
wersji plus nowo dolozony prostopadloscian). Skrypt przebudowujacy prefab powinien
najpierw sprzatnac stare zderzaki, a dopiero potem dodac swoj - inaczej po kilku
przebiegach obiekt zbiera ich kolekcje i zachowuje sie dziwnie przy noszeniu.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260713-1420-convex-meshcollider-swallows-hollow-interiors|Convex MeshCollider na skorupie połyka jej wnętrze - promienie nigdy nie trafią w części w środku]] - wspolne: convex, meshcollider, physics
- [[dynamic-rigidbody-no-nonconvex-meshcollider|Dynamic Rigidbody → Primitive or Convex Collider, Never Non-Convex Mesh]] - wspolne: rigidbody, meshcollider, physics
- [[20260714-2220-maxspeed-clamp-is-not-a-speed|maxSpeed to KLAMRA, nie prędkość - pojazd i tak stanie na (napęd / tłumienie)]] - wspolne: rigidbody, physics
- [[20260724-0907-arcade-car-climbs-walls|Arkadowe auto na Rigidbody wspina sie po scianach i odlatuje w niebo]] - wspolne: rigidbody, physics
- [[freeze-inertia-tensor-not-restored|FreezeAll + automaticInertiaTensor=false Zeroes the Inertia Tensor]] - wspolne: rigidbody, physics
- [[self-collision-compound-colliders-ignore|Self-Collision Compound BoxColliders → Physics.IgnoreCollision]] - wspolne: rigidbody, physics
<!-- /POWIAZANE:auto -->
