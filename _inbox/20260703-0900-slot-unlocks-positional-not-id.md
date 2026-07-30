---
title: Numer w id odblokowania ≠ pozycja slotu (sloty kupowane w dowolnej kolejności)
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-03'
project: Timber_Tycoon
tags:
- unity
- unlocks
- data-driven
- bitmask
- save-system
- economy
applies_to: []
source: ''
severity: major
suggested-category: engine/lessons
---

# Numer w id odblokowania ≠ pozycja slotu (sloty kupowane w dowolnej kolejności)

## Anti-pattern
System slotów (etaty załogi) z odblokowaniami na drzewku `slot_1..slot_5`, gdzie stan
per slot trzymany jest POZYCYJNIE (bitmaska 0..count−1, count = liczba kupionych id),
a handler zakupu wyprowadza indeks z NUMERU w unlockId (`slot_4` → bit 3). Drzewko NIE
wymusza kolejności zakupów (karteczki niezależne, bez pól prerequisite) — gracz kupuje
`slot_4` przed `slot_3` i bit z numeru ląduje POZA maską pozycyjną: opłata/aktywacja
idzie w martwy bit (pieniądze w błoto, „kupiony" pracownik nie przychodzi, a UI pokazuje
ostatni pozycyjny slot jako nieopłacony i pozwala zapłacić DRUGI raz).

## Dlaczego to podstępne
Testy naturalnie kupują sloty po kolei (1,2,3…) — bug nie wychodzi w smoke testach;
wyszedł dopiero w przeglądzie adwersaryjnym. Dwa systemy adresowania (id-suffix vs
pozycja) wyglądają identycznie, dopóki są kupowane rosnąco.

## Reguła
Przy stanie pozycyjnym indeks slotu po zakupie licz Z LICZBY posiadanych odblokowań
(`slot = count − 1`, jeśli zakup trafia do zbioru PRZED emisją eventu — sprawdź kolejność!),
nigdy z numeru w id. Alternatywa: wymusić kolejność zakupów (prerequisite w danych).
Do tego strażnik idempotencji na płatności per slot (`if (IsPaid(slot)) return true`).

## Transfer
Każdy data-driven system odblokowań z „ponumerowanymi" wpisami kupowanymi niezależnie
(sloty, tiery, piętra budynku). Test wykrywający: kup wpisy w ODWROTNEJ kolejności.
