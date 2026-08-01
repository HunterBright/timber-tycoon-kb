---
title: Pomiar proporcji z renderu klamie trzy razy, zanim zacznie mowic prawde
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-25'
project: Kerf - Sawmill Tycoon
tags:
- image-analysis
- measurement
- validation
- false-green
- character-reference
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# Pomiar proporcji z renderu klamie trzy razy, zanim zacznie mowic prawde

## Kontekst

Trzeba bylo zmierzyc "ile glow na wzrost" ma postac na wygenerowanym renderze,
zeby porownac kandydatow bez zdawania sie na oko. Wygladalo na piec minut roboty.
Zajelo cztery podejscia, a **kazde poprzednie podawalo liczbe, ktora wygladala wiarygodnie**.

## Cztery pulapki, po kolei

1. **Maska sylwetki lapie cien na podlodze i winiete tla.** Prog "odleglosc od koloru
   narozników > 26" wciagnal miekki cien pod stopami i przyciemnione brzegi kadru.
   Wzrost wyszedl zawyzony, wiec liczba glow - zanizona. **Lekarstwo:** wyzszy prog
   PLUS warunek nasycenia (cien i winieta sa szare, `sat < 18`; kazdy element stroju
   jest nasycony) PLUS odrzucenie plam dotykajacych krawedzi kadru.
2. **"Szyja jako najwezsze miejsce sylwetki" wypada na barkach.** W pozie T ramiona daja
   ogromny skok szerokosci; minimum szukane w gornych 40% wysokosci ladowalo w klatce.
   **Lekarstwo:** najpierw znalezc skok barkow (najwieksza roznica szerokosci miedzy
   wierszami), potem szukac zwezenia TYLKO nad nim.
3. **"Najszerszy wiersz glowy" to w istocie bark.** Bark jest szerszy od glowy, wiec
   szukanie maksimum "w obszarze glowy" trafialo w bark i cala dalsza logika sie sypala
   (kontrola "brak miejsca na szyje" przerwala prace - i dobrze).
4. **Broda z koloru skory wypada na brodzie zarostu**, a przy szerokiej szczece prog 50%
   szerokosci twarzy tnie za wysoko. Blad w druga strone niz punkty 1-3.

## Co uratowalo sprawe

**Kontrola na obrazku o znanym wyniku.** Jeden obrazek zmierzony wczesniej recznie
(glowa 31% wzrostu = 3,17 glowy) przepuszczany przez kazda wersje pomiaru natychmiast
pokazywal, ktora wersja klamie. Bez tego wszystkie cztery wersje "dzialaly".

**Nakladka na obrazek.** Wrysowanie wykrytych linii (czubek, szyja, bark) w obrazek
i **spojrzenie na niego** wykrylo blad nr 2 w kilka sekund. Sama liczba go nie wykrywala.

## Regula do zapamietania

Dwie niezalezne metody pomiaru daja przedzial, w ktorym lezy prawda: metoda
"czubek->szyja z sylwetki" **zawyza glowe** (dolicza szyje i czapke), metoda
"czubek->broda z koloru skory" **zaniza** ja. Kiedy obie zgadzaja sie co do kolejnosci
kandydatow, wybor jest bezpieczny, nawet jesli sama liczba jest niepewna.

Patrz tez: [[feedback_probe_must_be_able_to_fail]] - kontrola, ktora nie potrafi
zawiesc, jest bezwartosciowa. Tu kontrole przerwaly prace dwa razy i za kazdym razem
wskazaly prawdziwa przyczyne.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260722-1652-relative-only-test-blind-to-common-mode-error|Test porownujacy instancje MIEDZY SOBA jest slepy na blad wspolny (common-mode)]] - wspolne: false-green, measurement
<!-- /POWIAZANE:auto -->
