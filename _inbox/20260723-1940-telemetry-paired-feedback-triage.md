---
type: pattern
project: Kerf - Sawmill Tycoon
suggested-category: workflow/patterns
tags: [playtest, telemetry, triage, logging, tester-feedback]
date: 2026-07-23
status: draft
---

# Triage feedbacku testera w parze z telemetria i logiem

## Problem
Lista subiektywnych uwag testera to tylko polowa obrazu. Tester zglasza to, co CZUL,
a nie to, co system robil zle. Ocenianie sugestii bez danych konczy sie priorytetyzacja
"co brzmi najciekawiej" zamiast "co najbardziej boli".

## Wzorzec
1. Kazdy build testerski wypuszczaj z telemetria (JSONL, 1 event/linia) + zbieraj Player.log.
2. Triage robic ROWNOLEGLE trzema torami (osobni agenci/przebiegi):
   a) skan logu (bledy + LICZNIKI powtarzajacych sie linii),
   b) parsowanie telemetrii (sesje, progresja, ekonomia, czasy realizacji, eventy porazek),
   c) recon kodu pod wykonalnosc kazdej sugestii (pliki:linie + wycena S/M/L).
3. Dopiero potem triage: (A) tanie i bezdyskusyjne, (B) decyzje designowe wlasciciela wizji,
   (C) pomin z uzasadnieniem. Sugestie testera NIE sa rownowazne - wycena z reconu
   zmienia kolejnosc bardziej niz sila przekonania testera.

## Kluczowe odkrycie
Najwiekszy problem sesji (spawner aut zablokowany przez ~34% czasu gry) NIE BYL zgloszony
przez testera - wyszedl z POLICZENIA powtorek jednej linii logu (563x "brak wolnych slotow")
i skorelowania jej z eventami porazek w telemetrii (3x order_fail dokladnie o polnocy).
Spam-log to nie tylko szum do wyciszenia - to darmowy licznik problemow systemowych.
Przy wyciszaniu: NIE kasowac sygnalu, logowac RAZ NA EPIZOD + czas trwania
(wzorzec: if (wait==0) log("start"); ... if (wait>0) log($"koniec po {wait}s")).

## Anty-pulapka
Dokladajac dzwiek/alert do zdarzenia sprawdz, czy inny system NIE sygnalizuje juz tego
samego momentu (u nas: plakietka "Masz klienta!" pojawia sie w tej samej klatce, w ktorej
dzwoni dzwonek lady - drugi dzwiek bylby dublem).
