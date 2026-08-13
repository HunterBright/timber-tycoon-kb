---
title: Dlugie polecenia w Unity z wiersza polecen - zadania odczepione zamiast czekania na polaczenie
type: pattern
status: draft
confidence: medium
verified: 2026-08-13
tags: [unity, cli, agent, automatyzacja, build, pipeline]
date: 2026-08-13
project: GameDevOS
source: https://packages.unity.com/com.unity.pipeline
applies_to: [sterowanie-unity-z-agenta, build-automatyczny, testy]
---

# Dlugie polecenia w Unity: zadanie odczepione zamiast czekania na polaczenie

## Problem, ktory to rozwiazuje

Gdy agent steruje Edytorem Unity przez lokalny serwer HTTP, kazde dlugie
polecenie (budowanie, import assetow, pelny zestaw testow) rodzi ten sam
uklad: **polaczenie odpada po czasie, a praca w Edytorze leci dalej**. Agent
nie wie, czy zadanie sie udalo, czy padlo, i zwykle powtarza je od poczatku.

Drugi objaw tego samego problemu: **zajety Edytor wyglada na nieosiagalny**.
Zapytanie o stan czeka w kolejce za dlugim poleceniem, wiec automat uznaje,
ze Edytor umarl, i go restartuje - w srodku budowania.

## Wzorzec

**Rozdziel zlecenie od odbioru wyniku.**

1. Zlecenie zwraca **numer zadania natychmiast**, a nie wynik. Praca idzie
   w tle.
2. Wynik odbiera sie **osobnym zapytaniem po numerze**, dowolnie pozniej.
   Klient moze sie odpiac i wrocic.
3. Wyniki sa **przetrzymywane przez okreslony czas** (u Unity: godzina albo
   ostatnie sto zadan), zeby odbior nie musial byc natychmiastowy.
4. **Postep jest podawany osobnym kanalem**, obslugiwanym poza glownym watkiem,
   wiec odpowiada takze wtedy, gdy glowny watek stoi zablokowany.
5. **Zapytania o stan sa obslugiwane rownolegle** i nie czekaja za dlugim
   poleceniem. Same polecenia nadal ida pojedynczo, w kolejnosci zgloszen.
6. Anulowanie jest **dwustopniowe**: zadanie w kolejce ginie od razu, zadanie
   biegnace dostaje **prosbe o wspolprace** i samo sprawdza, czy ma przerwac.

## Dlaczego akurat tak

Trzy rzeczy, ktore latwo zrobic zle:

- **Nie wystarczy wydluzyc limit czasu.** Limit rozwiazuje przypadek "trwa
  dlugo", nie rozwiazuje "klient sie rozlaczyl". Numer zadania rozwiazuje oba.
- **Rownoleglosc dotyczy odpytywania, nie wykonywania.** Gdyby polecenia
  szly rownolegle, dwa naraz dotykalyby stanu Edytora. Rownolegle ma byc
  **pytanie o stan**, zeby zajety Edytor odpowiadal "pracuje", a nie milczal.
- **Zimny start to osobny stan.** Edytor po otwarciu projektu importuje
  i kompiluje. Polecenie wyslane w tym oknie nie powinno "zawiesc" - powinno
  dostac odpowiedz "jeszcze sie ukladam", odrozniona od "gotowy".

## Dowod

Ten wzorzec wszedl do `com.unity.pipeline` w wersji **0.5.0-exp.1**,
opublikowanej **12.08.2026** (odczytane z rejestru pakietow
`packages.unity.com`, pole `dist-tags.latest`; poprzednia wersja 0.4.0-exp.1
stala od 24.07.2026).

Konkretnie w dzienniku zmian tej wersji: zadania odczepione z odbiorem
po numerze i przetrzymywaniem wynikow, limit dlugiego wykonania podniesiony
**z 30 sekund do 24 godzin**, rownolegla obsluga zapytan przy zachowaniu
pojedynczego wykonywania polecen, osobny kanal postepu dzialajacy przy
zablokowanym glownym watku, oraz sygnal "jeszcze sie ukladam" przy zimnym
starcie projektu.

**Zastrzezenie:** to jest odczyt dziennika zmian producenta, **nie nasz pomiar
dzialania**. Wzorzec jest wart przepisania niezaleznie od tego, czy ta
konkretna paczka sprawdzi sie u nas.

## Kiedy to sie NIE oplaca

Przy poleceniach krotszych niz kilka sekund numer zadania to zbedny drugi
obieg. Wzorzec zaczyna sie oplacac tam, gdzie polecenie moze przezyc
polaczenie: budowanie, import duzych partii assetow, pelne testy.

## Czy to przeniesie sie na inny projekt

**Tak.** To jest ogolny wzorzec sterowania dowolnym programem z interfejsem
graficznym przez agenta, nie cecha Unity. Ten sam uklad pasuje do Blendera
uruchamianego bezokiennie przy ciezkich operacjach (remesh, wypalanie map,
symulacje) oraz do kazdego generatora modeli 3D liczacego minutami.

## Powiazane

- [[dzwignia-unity-cli]]
- [[20260808-sprawdzian-ktory-nie-umie-pasc]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260807-1155-skrypt-zarzadzany-blender-bezokienkowy|Skrypt zarzadzany - dyscyplina edycji Blendera bezokienkowo]] - wspolne: agent, automatyzacja, pipeline
- [[20260805-1520-przedawnienie-wiedzy-jest-funkcja-typu-wpisu|Przedawnienie wpisu jest funkcja jego TYPU, nie uplywu czasu]] - wspolne: agent, automatyzacja
- [[ANALIZA-ROZMOWY|Analiza rozmowy o automatyzacji pipeline'u - mocne i słabe strony]] - wspolne: automatyzacja, pipeline
- [[DZWIGNIA-UNITY-CLI|Dźwignia, która stoi nieużywana - własne komendy w Unity CLI]] - wspolne: automatyzacja, pipeline
<!-- /POWIAZANE:auto -->
