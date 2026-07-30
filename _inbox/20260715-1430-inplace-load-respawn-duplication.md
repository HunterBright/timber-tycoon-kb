---
title: Load "w miejscu" bez czyszczenia = respawnowane obiekty stackuja sie z kazdym wczytaniem
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-15'
project: Kerf - Sawmill Tycoon
tags:
- unity
- save-load
- in-place-load
- respawn
- duplication
- scene-reload
- isaveable
- idempotency
applies_to:
- unity
- save-system
- runtime-spawned-objects
source: ''
severity: high
time_lost: ~1h diagnoza (log + 3 systemy trwalosci)
suggested-category: engine/lessons
---

# Load "w miejscu" bez czyszczenia = respawnowane obiekty stackuja sie z kazdym wczytaniem

## Problem
Gracz zglosil "zdublowane drzewa" - wiele obiektow w JEDNYM punkcie. Log pokazal te same wspolrzedne
respawnowane po KAZDYM "Wczytaj gre": N wczytan = N+1 kopii kazdego runtime-spawnowanego obiektu w tym
samym miejscu. Nie bylo to bugiem spawnu/sadzenia - tylko wczytywania.

## Root cause
Wczytanie zapisu bylo robione IN-PLACE (bez przeladowania sceny): SaveManager rozdawal dane do juz
istniejacych ISaveable, a osobny rejestr (WorldSpawnRegistry) w LoadSaveData szedl PROSTO do petli
Instantiate - NIGDY nie niszczyl/nie czyscil zywych obiektow ze swoich list przed respawnem. Zywy swiat
+ respawn z save = podwojenie. Kazde kolejne wczytanie dokladalo warstwe.

Kluczowa subtelnosc (dlaczego "przeladuj scene" NIE jest tu poprawne): obiekty maja MIESZANA trwalosc.
Czesc (rosnace drzewa, klody, pniaki) jest RESPAWNOWANA z rejestru. Ale obiekty ktore "dorosly" w
runtime staly sie osobnymi ISaveable (ChoppableTree) rejestrowanymi w SaveManager i wczytujacymi stan
W MIEJSCU - ich NIKT nie respawnuje. Gdyby load przeladowywal scene, te runtime-spawnowane dorosle
zniknelyby na zawsze (nie ma ich w swiezej scenie, a SaveManager tylko aplikuje dane do istniejacych
obiektow, nie tworzy nowych). Wiec "reload sceny" naprawilby duplikaty, ale skasowalby dorosle drzewa.

## Solution
Uczynic respawn IDEMPOTENTNYM przy zachowaniu load-in-place: na POCZATKU LoadSaveData zniszczyc i
wyczyscic wszystkie zywe obiekty ze WSZYSTKICH list rejestru PRZED petla Instantiate. Wtedy load w
miejscu odtwarza swiat raz, a obiekty wczytywane w miejscu (ISaveable) dostaja stan bez dublowania.

Wzorzec: kazdy runtime-rejestr obiektow spawnowanych z save musi kasowac swoja zywa zawartosc przed
respawnem (albo respawnowac tylko brakujace po kluczu). Load bez tego zaklada "swieza scena" - ale jesli
load jest in-place, to zalozenie jest falszywe.

## What didn't work
- Trop "podwojny event/log przy sadzeniu" - falszywy. Jeden obiekt logowal init dwa razy (raz po
  Instantiate, raz w Start) - to nie druga instancja. Lekcja: policz PRAWDZIWE obiekty (po Awake/Start
  rejestru), nie linie logu.
- "Przeladuj scene przy load" - kuszace i czystsze architektonicznie, ale w systemie z MIESZANA
  trwaloscia (respawn z rejestru + in-place ISaveable dla runtime-spawnowanych) skasowaloby obiekty,
  ktorych nikt nie odtwarza. Najpierw zrozum WSZYSTKIE sciezki trwalosci, dopiero potem wybierz reload
  vs idempotentny respawn.

## Transferability
Kazdy projekt z zapisem, ktory: (a) spawnuje obiekty w runtime i respawnuje je z save przez wlasny
rejestr, ORAZ (b) wczytuje bez przeladowania sceny (quick-load / continue). Klasyczny podwojny-respawn.
Regula: respawn z save MUSI byc idempotentny (kasuj-przed-odtworz albo dedup po kluczu), zwlaszcza gdy
load nie przeladowuje sceny.

## Related
- Wielosystemowa trwalosc (3 niezalezne systemy zapisu drzew) - zrodlo "luk"
- Rodzina "Editor lies" / build-smoke jako straznik regresji
