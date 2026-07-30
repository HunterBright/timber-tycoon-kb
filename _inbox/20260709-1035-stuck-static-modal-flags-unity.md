---
title: Zawieszone statyczne flagi modalne blokuja interakcje na stale
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-09'
project: Timber_Tycoon
tags:
- unity
- coroutines
- statics
- ui
- input-blocking
- self-heal
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Zawieszone statyczne flagi modalne blokuja interakcje na stale

## Objaw
Po przerwanej minigrze/oknie gracz traci na stale czesc interakcji (np. prompt sadzenia
nigdy sie nie pokazuje) albo trzymane narzedzie robi sie niewidzialne, choc logika dziala.

## Przyczyna (transferowalna)
Wzorzec "globalna flaga + para set/unset w przebiegu":
- statyczna flaga (`buryingActive`, `isOpen`, `viewmodelHidden`) ustawiana na poczatku
  przebiegu i zdejmowana na koncu,
- przebieg realizowany coroutina lub sekwencja UI.

Jesli obiekt-wlasciciel zginie lub zostanie wylaczony W TRAKCIE (Destroy, SetActive(false),
wyjatek, pominieta sciezka cancel), coroutine umiera bez `finally`, flaga zostaje TRUE
na zawsze. Konsumenci (np. centralny Update interakcji, ktory chowa wszystkie prompty gdy
"cos jest otwarte") sa zablokowani globalnie. Przy wylaczonym domain reload statics
przezywaja tez miedzy sesjami Play w edytorze.

## Fix (3 warstwy, do reuzycia)
1. Flaga statyczna ma WLASCICIELA: `static Owner buryingOwner;` ustawiany razem z flaga.
   W OnDisable/OnDestroy wlasciciela: `if (owner == this) flaga = false`.
2. `[RuntimeInitializeOnLoadMethod(SubsystemRegistration)]` zeruje statics (domain reload off).
3. Samonaprawa u konsumenta stanu: jesli stan "aktywny", ale zrodlo faktycznie nie dziala
   (canvas zgasl / zadna minigra-IsActive nie trwa) przez N klatek - uzgodnij stan
   (np. ToolViewmodel przywraca schowane narzedzie po ~30 klatkach bez aktywnej minigry).

Warstwa 3 jest najcenniejsza: pokrywa KAZDA przyszla przyczyne (nowa minigra, wyjatek),
bez latania kazdego wywolujacego osobno.

## Anty-wzorzec
Naprawianie tylko jawnych sciezek cancel. Zawsze zostanie sciezka, ktorej nie ma
(wyjatek, Destroy z zewnatrz, scene unload).
