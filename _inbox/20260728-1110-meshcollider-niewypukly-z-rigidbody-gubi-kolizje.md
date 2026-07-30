---
title: Zderzak z pelnej siatki na obiekcie z fizyka = obiekt znika ze swiata
type: lesson
status: draft
confidence: low
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
suggested-category: engine/lessons
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
