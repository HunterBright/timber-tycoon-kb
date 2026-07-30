---
title: '"Ile procent kadru, tyle procent wysokości" to nieprawda, gdy obiektyw patrzy z góry'
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-27'
project: Kerf - Sawmill Tycoon
tags:
- pomiar
- perspektywa
- kamera
- referencje
- obraz
- proporcje
applies_to: []
source: ''
suggested-category: engine/lessons
---

# "Ile procent kadru, tyle procent wysokości" to nieprawda, gdy obiektyw patrzy z góry

## Objaw

Wszystkie wysokości mierzone ze zdjęcia referencyjnego były liczone tak:
znajdź podeszwę i czubek głowy, a potem wysokość dowolnego punktu to jego
położenie między nimi, liczone liniowo. Kontrole przechodziły, sylwetka się
zgadzała, nic nie zgrzytało.

Dopiero test na renderze WŁASNEGO modelu — takiego, w którym wysokość każdego
pierścienia znałem co do milimetra — pokazał systematyczne przesunięcie
o 1,7% wzrostu (29 mm przy 1,75 m), z maksimum dokładnie w połowie wysokości
i zerem na obu końcach.

## Przyczyna

Rzut perspektywiczny odwzorowuje wysokość na wiersz obrazu funkcją
homograficzną, a nie liniową. Gdy oś obiektywu jest równoległa do podłogi,
nieliniowość znika i przybliżenie liniowe jest dokładne. Wystarczy jednak
pochylić kamerę o kilka stopni, żeby pojawiło się wygięcie.

Zmierzone dla sześciu ujęć tej samej postaci (odległość ~3 m, wzrost 1,75 m):

| pochylenie kamery | wygięcie w połowie wysokości |
|---|---|
| 2° | 10 mm |
| 6° | 29 mm |
| 25° | 124 mm |
| 38° | 206 mm |

**Dlaczego to tak długo nie wychodzi:** błąd jest zerowy na obu końcach
zakresu, bo końce są punktami zaczepienia skali. Każda kontrola porównująca
całą sylwetkę z całą sylwetką przechodzi. Widać to dopiero, gdy istnieje
niezależna prawda o wysokości czegoś W ŚRODKU.

## Druga warstwa: głębokość

Sama nieliniowość to nie wszystko. Przy kamerze patrzącej z góry punkt bliżej
obiektywu rzutuje się NIŻEJ niż punkt na tej samej wysokości, ale dalej.
Zmierzone przy pochyleniu 6°: 10 cm w stronę obiektywu to 29 mm w dół przy
podłodze i 12 mm w GÓRĘ na wysokości klatki piersiowej (znak się odwraca
w miejscu, na które kamera patrzy wprost).

Praktyczny skutek: najniższy piksel sylwetki to czubek buta, wysunięty do
przodu — więc „podeszwa" wypada na wysokości −59 mm, czyli pod podłogą.
Zaczepienie skali na skrajnych pikselach sylwetki bierze ten błąd na wejściu.

## Rozwiązanie

Mając zmierzoną kamerę, wysokość liczy się przez ODWRÓCENIE tego samego
rzutu, którego używa reszta kodu (nie osobnym wzorem — dwie kopie tej samej
arytmetyki rozjadą się prędzej czy później). Prosta bisekcja po wysokości
wystarcza i jest odporna na pomyłki w wyprowadzeniu wzoru.

Efekt na tym samym obrazku: błąd z 30,6 mm na 12,7 mm, a w środku ciała
na 2 mm. Reszta to głębokość — żeby ją zdjąć, linijka musi dostać głębokość
mierzonego punktu, a nie zakładać, że leży na osi.

## Reguła ogólna

Jeśli wyciągasz proporcje z renderu albo zdjęcia i nie znasz kamery, wolno
zakładać liniowość tylko wtedy, gdy oś obiektywu jest pozioma. W każdym innym
przypadku najpierw rozwiąż kamerę, a potem mierz — inaczej kupujesz sobie
błąd rzędu procenta, który jest niewidoczny dla wszystkich kontroli
porównujących obraz z obrazem.
