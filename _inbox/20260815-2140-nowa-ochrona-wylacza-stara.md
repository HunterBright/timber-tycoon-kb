---
title: Dodanie pliku do zbioru pilnowanego przez jedną bramkę potrafi wyłączyć drugą bramkę, która opierała się na jego nieobecności
type: lesson
status: draft
confidence: high
verified: '2026-08-15'
date: 2026-08-15
project: Another Quest
tags:
- gates
- tooling
- regression
- state-integrity
- adversarial-review
applies_to: []
source: 'LOCK CUR 2026-08-15, tools/canon_manifest.py + tools/canon_apply.py'
severity: high
suggested-category: engine/lessons
time_lost: '~40 min przeglądu adwersaryjnego; bez niego dziura weszłaby do repo'
---

# Dodanie pliku do zbioru pilnowanego przez jedną bramkę potrafi wyłączyć drugą bramkę, która opierała się na jego nieobecności

## Problem

Narzędzie A (manifest odcisków) dostało nową klasę plików: dokumenty spoza katalogu kanonu, zbierane obok
niego i pilnowane tak samo jak moduł. Zmiana wyglądała na czysto addytywną — „pilnujemy o jedną rzecz więcej".

Narzędzie B (zapis do kanonu) miało bezpiecznik: **odmawiało zapisu do pliku, który nie ma wpisu w manifeście.**
Ten bezpiecznik chronił nowy plik *przypadkiem* — bo plik nie był w manifeście. Po dodaniu klasy wpis się
pojawił i bezpiecznik przestał działać: polecenie, które przedtem kończyło się odmową, zaczęło zapisywać
do świeżo zamrożonego dokumentu i zostawiać bramkę **zieloną**.

Gorzej: ślad audytowy zapisywał wtedy „bezpiecznik był włączony i to przepuścił". Przedtem ta sama operacja
wymagała jawnej flagi wymuszenia, która zostawiała w dzienniku `bezpiecznik_pominiety: true`. Czyli nie
„pojawiła się nowa droga", tylko **zniknęła ostatnia bariera i zepsuł się zapis o tym, że zniknęła.**

Druga sztuka tego samego rodzaju: narzędzie B po każdej swojej operacji odświeżało manifest, przeliczając
odciski **wszystkich** pilnowanych plików. Dla katalogu, którym B zarządza, to jest sens tej operacji.
Dla nowo dodanej klasy to była **pralka**: cicha edycja zamrożonego pliku zapalała bramkę na czerwono,
a najbliższa rutynowa operacja B na czymś zupełnie niezwiązanym gasiła zgłoszenie bez śladu.

## Root cause

**Bezpiecznik B pytał o obecność wpisu, a nie o uprawnienie.** „Nie ma cię w rejestrze" znaczyło u niego
„nie wolno cię ruszać" — co jest prawdą tylko dopóki rejestr pokrywa się z listą rzeczy, do których wolno
pisać. Nowa klasa rozerwała tę tożsamość: rejestr zaczął zawierać rzeczy, do których pisać **nie wolno
tym bardziej**.

Ogólna postać: **dwa mechanizmy ochronne dzieliły jedną strukturę danych, ale wyciągały z niej przeciwne
wnioski.** Dopóki zbiór był jednorodny, nikt tego nie zauważył. Pierwsza niejednorodna pozycja odwróciła
znaczenie u jednego z nich.

## Solution

1. **Bezpiecznik pytający „czy jesteś w rejestrze" zamień na pytający „czy wolno tu pisać".** Konkretnie:
   jawna kontrola zakresu (cel musi leżeć w katalogu, którym to narzędzie zarządza). W tym repo bliźniacza
   operacja *miała* taką kontrolę od początku — jedna z dwóch dróg zapisu jej nie dostała i przez lata
   nikt tego nie widział, bo osłaniał ją skutek uboczny.
2. **Operacja, która odświeża cały rejestr „przy okazji", musi mieć wyłącznik dla klas, których nie
   dotyczy.** Baseline chronionego pliku wolno przesuwać **wyłącznie** jawnym poleceniem z podanym powodem.
3. **Dźwignie trybu porażki nie mogą być dostępne dla trybu produkcyjnego.** Ta sama flaga, która w teście
   dowodzi, że bramka mierzy, w prawdziwym uruchomieniu zwracała zielone zero na wyłączonym porównaniu.
4. **Autotest musi przechodzić przez warstwę CLI, nie tylko wołać funkcję.** Wszystkie próby wołały funkcję
   sprawdzającą wprost i podawały dźwignie jawnie, więc **jedna dopisana linijka w `main()` wyciszała ochronę
   przy zielonym autoteście.** Cała warstwa „co CLI przekazuje dalej" leżała poza testem.

## What didn't work

- **Rozumowanie „to zmiana czysto addytywna, nic nie psuje".** Zbiór pilnowany przez jedno narzędzie jest
  wejściem dla drugiego. Powiększenie zbioru to zmiana wejścia, nie dodatek.
- **Lista pilnowanych plików scalana z pliku stanu i z kodu (suma).** Wyglądało na odporne („kodu nie da
  się wyciszyć danymi"), ale ścieżka raz zapisana do stanu **nigdy z niego nie wypadała**, więc usunięcie
  wpisu z kodu nie odmrażało pliku, a przeniesienie pliku zostawiało bramkę czerwoną na zawsze. Docstring
  obiecywał coś, czego kod nie robił. Naprawa: **kod jest jedynym źródłem, stan dostaje tylko kopię do wglądu.**
  Wtedy dane nie mogą wpłynąć na listę w żadną stronę, a odmrożenie jest dwustopniowe i widoczne.

## Transferability

Nie ma tu nic specyficznego dla silnika ani gatunku. Wzorzec występuje wszędzie, gdzie **kilka mechanizmów
ochronnych czyta jeden rejestr**: lockfile i CI, allowlista i firewall, migracje i checksumy schematu,
`.gitignore` i skrypt czyszczący. Pytanie do zadania przy każdym powiększeniu chronionego zbioru:

> **Kto jeszcze czyta ten zbiór — i czy dla któregoś z nich „jest na liście" znaczy coś przeciwnego
> niż dla mnie?**

Drugi wniosek jest jeszcze szerszy: **ochrona, która działa przez skutek uboczny (bo pliku akurat nie ma
w rejestrze), jest niewidzialna w kodzie i znika bez ostrzeżenia.** Jeśli plik ma być chroniony, kontrola
musi to mówić wprost.

## Related

- [[20260815-0840-bezpiecznik-lapie-narzedzia-nie-drogi|Bezpiecznik wymienia narzedzia, a zagrozenie chodzi drogami - poszerz wszystkie matchery naraz]] - wspolne: bramki, ochrona, matchery
- [[20260814-0930-dzwignia-zepsuj-musi-nazwac-winowajce|Dzwignia --zepsuj musi nazwac winowajce, nie tylko podniesc kod wyjscia]] - wspolne: bramki, redproof
- [[20260815-2145-autotest-omijajacy-warstwe-cli|Autotest wolajacy funkcje zamiast CLI zostawia warstwe argumentow bez pokrycia]] - wspolne: bramki, testy
