---
title: Pre-save flush dla systemow automatyzacji mutujacych zapisywany swiat
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- save-system
- automation
- npc
- coroutine
- timescale
- race-condition
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Pre-save flush dla systemow automatyzacji mutujacych zapisywany swiat

## Problem

System automatyzacji (NPC-pracownik, auto-harvest, kolejka produkcji) prowadzi WIELOKLATKOWE
operacje na obiektach swiata (fizyczna animacja, sekwencja spawnow). Jesli zapis gry trafi
w srodek takiej operacji, plik zapisu widzi stan posredni, ktorego system wczytywania nie
umie odtworzyc (w Timber Tycoon: runtime-dorosle drzewo poza stanem Standing nie mialo
respawnera - drzewo znikalo bez lupow). Autosave co N minut + dlugie zlecenia = trafienie
niemal pewne, nie teoretyczne (~40% okna czasowego przy 50-drzewnym zleceniu).

## Wzorzec

1. SaveManager wystawia synchroniczny event `OnBeforeSave`, wolany na POCZATKU Save(),
   przed zbieraniem danych od ISaveable.
2. System automatyzacji subskrybuje i FLUSHUJE biezaca operacje W JEDNEJ KLATCE:
   - StopCoroutine wszystkich uchwytow (takze WEWNETRZNYCH coroutine - zatrzymanie
     zewnetrznej NIE zatrzymuje zagniezdzonej odpalonej przez StartCoroutine!),
   - analityczne dokonczenie stanu (pozycje/kierunki liczone wzorem zamiast animacji),
   - synchroniczne dokonczenie efektow (spawny, zuzycie zasobow, zaliczenie sztuki),
   - sprzatniecie zasobow trzymanych przez coroutine (zapetlone SFX! - uchwyt trzeba
     wystawic w polu, lokalna zmienna coroutine jest nieosiagalna).
3. TWARDY wymog: flush ma ZERO yield - zapis z menu pauzy leci przy timeScale=0,
   wiec kazde czekanie na klatke/sekundy nigdy nie wroci.
4. Wznowienie po wczytaniu: LoadSaveData = TYLKO deserializacja (StartCoroutine w
   LoadSaveData wykonuje body synchronicznie do pierwszego yielda - swiat moze byc
   w polowie odtwarzania); start petli dopiero na `OnAfterLoad`, a obiekty swiata
   wybierane SWIEZO przy kazdym kroku (zadnych cache'owanych referencji miedzy save/load).

Efekt: plik zapisu zawsze widzi stan "miedzy jednostkami pracy" - klasa bledow
"obiekt w locie znika po wczytaniu" jest niemozliwa konstrukcyjnie.

## Pulapka testowa (sonda/test automatyczny)

Asercje wznowienia po in-place load robic NATYCHMIAST po powrocie Load() (ta sama klatka,
petla wznowiona na OnAfterLoad jeszcze nie tykala). Czekanie "zeby sie ustabilizowalo"
przegrywa wyscig z DZIALAJACYM systemem: na przyspieszonym tiku testowym drwal zdazyl
DOKONCZYC cala kolejke, zanim sonda po 4 s spojrzala na licznik - falszywy FAIL mimo
poprawnego dzialania.

## Kiedy stosowac

Kazdy system, ktory (a) mutuje obiekty ISaveable wieloklatkowo i (b) moze byc aktywny
w momencie zapisu (autosave!). Nie dotyczy operacji jednoklatkowych ani czysto
wizualnych (nie zapisywanych).
