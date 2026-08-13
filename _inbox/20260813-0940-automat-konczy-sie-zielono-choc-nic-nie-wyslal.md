---
title: Automat konczy sie zielono, choc od tygodnia nic nie wyslal
type: lesson
status: draft
confidence: high
verified: '2026-08-13'
date: 2026-08-13
project: GameDevOS
tags:
- automatyzacja
- ci
- git
- bramki
- harmonogram
applies_to: []
source: 'D:\GameDevOS\tools\publikuj.ps1, D:\GameDevOS\.gitignore'
severity: high
suggested-category: workflow/lessons
time_lost: 'tydzien niezauwazonej pracy, okolo 40 minut diagnozy'
---

# Automat konczy sie zielono, choc od tygodnia nic nie wyslal

## Problem

Codzienny automat (raport z rynku, potem pielegnacja bazy wiedzy, potem commit
i wysylka na GitHuba) chodzil przez tydzien i **kazdego dnia konczyl sie sukcesem**:
zadanie w harmonogramie zwracalo kod 0, pliki raportow lezaly na dysku z aktualnymi
datami, logi wygladaly normalnie.

Ostatni commit w repozytorium byl sprzed siedmiu dni. W drzewie roboczym czekalo
**104 niezacommitowanych zmian**, w tym siedem dziennych raportow. Nikt tego nie
zauwazyl, bo wszystko, na co patrzylo sie na co dzien - pliki raportow - bylo na miejscu.

## Root cause

Dwie przyczyny, jedna pod druga.

**Blizsza:** bramka sekretow slusznie zatrzymywala commit. Zwiad odkladal zrzuty
robocze z GitHuba w KORZENIU repozytorium (`.tmp_scout/`, `tmp_arxiv/`, `mkt.json`),
a taki zrzut zawiera adresy klonowania w postaci `git@<serwer>`, ktore detektor
czyta jako adresy e-mail. Umowa „plik roboczy zaczyna sie od podkreslnika" i odpowiadajaca
jej regula pomijania obowiazywaly **tylko wewnatrz `tools/`**, wiec katalog roboczy
zalozony pietro wyzej nie byl objety niczym.

**Glebsza, i to jest wlasciwa lekcja:** skrypt uruchamiajacy sprawdzal wynik
kazdego kroku poza ostatnim. Wywolywal `publikuj.ps1` i konczyl sie, nie patrzac
na to, co ten zwrocil. Kod wyjscia calego przebiegu opisywal wiec „czy sie nie
wywrocilem", a nie „czy praca gdziekolwiek dotarla".

## Solution

1. Regula pomijania poszla **za umowa, nie za lista nazw**, i objela takze korzen:
   `.tmp_*/`, `tmp_*/` plus konkretne zrzuty w korzeniu. Z 38 znalezisk bramki
   zostaly 3.
2. Sama umowa trafila do instrukcji agenta: **pliki robocze pisz do `tools\_*`,
   nigdy do korzenia**. Bez tego regula pomijania goni agenta, zamiast go wyprzedzac.
3. Kazdy skrypt uruchamiajacy sprawdza teraz `$LASTEXITCODE` po kroku publikacji
   i wypisuje glosne ostrzezenie do logu, gdy commit nie powstal.

## What didn't work

Szukanie przyczyny w logach automatu. Log konczyl sie na przedostatnim kroku
i wygladal zdrowo, bo ostatni krok w ogole do niego nie pisal. Diagnoza ruszyla
dopiero od `git log` i `git status`, czyli od **pomiaru skutku, a nie przebiegu**.

## Transferability

Dotyczy kazdego automatu z bramka na koncu lancucha, niezaleznie od jezyka
i projektu: CI, ktory nie publikuje artefaktu, skrypt kopii zapasowych,
ktory nie odklada pliku, eksport assetow, ktory nie dochodzi do repozytorium.

Dwa wnioski do przeniesienia:

- **Dowodem wykonanej pracy jest skutek, nie kod wyjscia procesu.** Sprawdzaj
  istnienie commita, artefaktu, pliku - a nie to, ze nic sie nie wywrocilo.
- **Dokladajac krok na koncu lancucha, dolóż razem z nim sprawdzenie jego wyniku.**
  Krok bez sprawdzenia jest cichy dokladnie wtedy, gdy zawodzi.

## Related

- [[20260813-0950-rozdziel-pomiar-od-publikacji]]
