---
title: Nieobecnosc w jednym kanale nie jest nieobecnoscia w swiecie
type: lesson
status: verified
confidence: high
verified: '2026-08-10'
tags: [weryfikacja, zwiad, metodyka, falszywy-negatyw]
date: '2026-08-10'
project: GameDevOS
suggested-category: workflow/lessons
source: https://claude.com/blog/auto-mode-default-in-claude-code
applies_to: [radar, weryfikacja-faktow, sledzenie-narzedzi]
severity: high
time_lost: doba opoznienia przy zmianie z terminem
---

# Nieobecnosc w jednym kanale nie jest nieobecnoscia w swiecie

## Objaw

9 sierpnia 2026 zwiad zglosil, ze narzedzie, ktorego uzywamy codziennie,
zmienia domyslny tryb dzialania. Sprawdzilem **dziennik zmian** tego narzedzia,
przeczytalem calosc, nie znalazlem ani slowa i **zapisalem w raporcie, ze
plotka sie nie potwierdza**.

10 sierpnia okazalo sie, ze zmiana jest prawdziwa, ogloszona **trzy dni
wczesniej**, ma **konkretna date wejscia w zycie za cztery dni** i dotyczy
wszystkich sesji interaktywnych. Ogloszenie poszlo **blogiem producenta**.

## Przyczyna

Bledem nie bylo sprawdzenie dziennika zmian. Dziennik zmian byl wlasciwym
miejscem do sprawdzenia i **odpowiedz stamtad byla prawdziwa**: tej zmiany
tam faktycznie nie ma.

Bledem bylo **uznanie jednego kanalu za rozstrzygajacy przy twierdzeniu, ktore
moglo wyjsc kilkoma kanalami naraz**, i zapisanie wyniku slowem „nie potwierdza
sie" zamiast „nie ma tego w dzienniku zmian".

Mechanizm, ktory to napedza: **przy szukaniu czegos nowego brak trafienia jest
przyjemny**, bo konczy watek i pozwala isc dalej. Przy szukaniu potwierdzenia
brak trafienia wymaga decyzji, a najtansza decyzja to uznac sprawe za zamknieta.

## Rozwiazanie

**Zanim napiszesz „nie potwierdza sie", wypisz kanaly, w ktorych ta konkretna
wiadomosc mogla sie ukazac, i policz, ile z nich odwiedziles.**

Minimalne listy, ktore sie u nas sprawdzily:
- **producent oprogramowania**: dziennik zmian, blog, dokumentacja, a przy
  zmianach zachowania takze ciagi znakow w samym programie;
- **model AI**: karta modelu, rejestr posrednika, komunikat firmy, rejestr
  areny;
- **narzedzie spolecznosci**: repozytorium, **sklep wlasciwy dla tego
  narzedzia**, rejestr pakietow.

I zapisuj wynik doslownie: **„nie ma tego w dzienniku zmian"** zamiast
**„to nieprawda"**. Pierwsze jest pomiarem, drugie wnioskiem, ktorego pomiar
nie uzasadnia.

## Co NIE zadzialalo

- **Czytanie dziennika zmian dokladniej.** Przeczytalem go w calosci,
  pol miliona znakow. Dokladnosc nie naprawia zlego kanalu.
- **Zaufanie ocenie wiarygodnosci zwiadu.** Zwiad podal to jako „srednia
  wiarygodnosc, prosze sprawdzic" i **mial racje**, a moje sprawdzenie bylo
  gorsze niz jego przypuszczenie.

## Dowod

Wpis producenta oddaje 559 735 bajtow, a **kontrolna zmyslona sciezka na tej
samej domenie daje 404**, wiec adres naprawde odroznia wpis istniejacy od
nieistniejacego. W osadzonych danych strony stoi data publikacji trzy dni
wczesniejsza niz moje sprawdzenie.

## Ten sam blad w innym przebraniu, wczesniej

To jest **drugie wystapienie tego samego ksztaltu w ciagu czterech dni**.
Wczesniej przez piec przebiegow z rzedu zapisywalismy, ze pewnej kategorii
narzedzi „nie ma". Sprawdzalismy dwa duze katalogi kodu. Odpowiedz lezala
w **sklepie z dodatkami wlasciwym dla tego programu**, ktorego nie
przeszukalismy ani razu, i lezala tam od kwietnia.

**Wniosek ogolny: „nie znalazlem" znaczy tyle, ile warte sa przeszukane
kanaly, i nigdy wiecej.**

## Czy to przeniesie sie na inny projekt

Tak, i jest to jedna z tych lekcji, ktore nie maja nic wspolnego z konkretnym
silnikiem ani jezykiem. Dotyczy kazdej sytuacji, w ktorej ktos raportuje brak:
brak bledu w logach, brak zgloszenia, brak wpisu w dokumentacji. **Pytanie
brzmi zawsze tak samo: ile kanalow sprawdziles i czy ten, ktory pominales,
nie jest wlasnie tym wlasciwym.**

## Powiazane
- [[20260810-0930-migawka-zamiast-zapytania-gdy-kanal-nie-ma-daty-zmiany]]
