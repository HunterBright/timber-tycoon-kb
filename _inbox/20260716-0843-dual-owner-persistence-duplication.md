---
type: anti-pattern
project: Timber_Tycoon
suggested-category: engine/anti-patterns
tags: [unity, save-system, persistence, duplication, single-owner]
date: 2026-07-16
status: draft
---

# Dwoch wlascicieli trwalosci jednego obiektu w save (rejestr + rekonstrukcja stanu)

## The trap
System zapisu ewoluuje: najpierw obiekt-rodzic (np. drzewo) odtwarza swoj stan po wczytaniu
i "przy okazji" spawnuje obiekty potomne (sadzonka). Potem powstaje centralny rejestr
runtime-spawnow, ktory TEZ zapisuje i odtwarza te obiekty. Nikt nie usuwa starego spawnu
z rekonstrukcji - "przeciez dziala".

## Why it fails
Przy wczytaniu obaj wlasciciele odtwarzaja obiekt: rejestr respawnuje zapisany egzemplarz,
a rekonstrukcja rodzica spawnuje drugi. Kazdy cykl save/load = +1 kopia (obie rejestruja
sie w rejestrze, wiec nastepny save ma juz 2 wpisy). Bonus-bledy: rodzic spawnuje obiekt
nawet jesli gracz zabral go przed zapisem (dupy z powietrza) i na sztywnym offsecie zamiast
zapisanej pozycji. Sprzatanie rejestru (DestroyAllTrackedSpawns) NIE pomaga, bo spawn
rekonstrukcji wykonuje sie PO nim w tym samym przebiegu load, a rejestracja potomka
(w Start) dopiero w nastepnej klatce.

## Symptoms
- +1 kopia obiektu po KAZDYM wczytaniu w trakcie sesji (in-place load), rosnie liniowo.
- Obiekt wraca po wczytaniu, mimo ze gracz go zebral przed zapisem.
- Obiekt po wczytaniu w innym miejscu niz zostawiony.
- W nazwach instancji slady dwoch zrodel (np. `X_RESPAWN` vs `X_Reconstructed`).

## Correct approach
JEDEN wlasciciel trwalosci per typ obiektu, zapisany wprost w komentarzu obu miejsc.
Jesli rejestr przejmuje trwalosc typu, spawn w rekonstrukcji rodzica trzeba USUNAC w tym
samym commicie (grep po nazwach `*_Reconstructed`/`Spawn*` w metodach Reconstruct*).
Do tego test repro w buildzie: doprowadz stan -> save -> in-place load -> licznik obiektow
MUSI byc rowny sprzed zapisu (test najpierw CZERWONY na starym kodzie, potem zielony).
