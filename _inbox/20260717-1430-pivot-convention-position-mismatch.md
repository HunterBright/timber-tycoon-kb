---
title: Pozycja zapisana dla jednego pivota psuje sie cicho przy podmianie assetu z innym pivotem
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-17'
project: Timber_Tycoon
tags:
- pivot
- placeholder
- position
- ground-snap
- raycast
- npc
- unity
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Pozycja zapisana dla jednego pivota psuje sie cicho przy podmianie assetu z innym pivotem

## Symptom
Po podmianie kostki-placeholder (pivot w SRODKU bryly) na docelowy model postaci
(pivot w STOPACH) NPC lewituje ~1 m nad ziemia. Zadna bramka techniczna tego nie lapie
(build/sondy zielone) - wychodzi dopiero w playtescie.

## Korzen
Zapisana pozycja (w SO/scenie) NIE jest "pozycja obiektu na ziemi", tylko "pozycja PIVOTA".
Y ustawione recznie dla placeholder-kostki znaczy "srodek ciala na wysokosci Y". Model
z pivotem w stopach postawiony na tym samym Y ma stopy tam, gdzie kostka miala brzuch.
Skala/rozmiar nie ma nic do rzeczy - blad jest w konwencji punktu odniesienia.

## Reguly
1. Przy KAZDEJ podmianie placeholdera na docelowy asset sprawdz konwencje pivota obu
   (srodek vs spod vs inne) - zapisane pozycje dziedzicza konwencje STAREGO assetu.
2. Dla obiektow "stojacych na ziemi" (NPC, propy) nie przepisuj Y recznie - zrob
   RUNTIME GROUND-SNAP: raycast w dol z pozycji + kilkadziesiat cm, stopy na hit.point.y
   z 1-2 mm wbicia. Odporny na przyszle przesuniecia i zmiany terenu.
   Trik: promien startujacy WEWNATRZ wlasnego collidera nie trafi go (scianki jednostronne),
   wiec nie trzeba wylaczac collidera na czas pomiaru.
3. Do smoke-testu dopisz check kontaktu z gruntem (spod boundsow renderera vs raycast
   w dol, tolerancja np. [-25, +10] cm) - lewitacja i zakopanie to ta sama klasa bledu.

## Powiazane
Ta sama rodzina co "origin na koncu klody wpadal pod teren przy przechyle" (Birch_Log
re-center) - pozycjonowanie zalezne od konwencji originu/pivota assetu.
