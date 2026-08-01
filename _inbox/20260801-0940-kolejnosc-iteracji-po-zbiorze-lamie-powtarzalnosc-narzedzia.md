---
title: Narzedzie masowe, ktore iteruje po zbiorze, przepisuje te same pliki w kolko
type: lesson
status: draft
confidence: high
verified: 2026-08-01
tags: [automatyzacja, python, determinizm, narzedzia, powtarzalnosc, git]
date: 2026-08-01
project: GameDevOS
suggested-category: workflow/general
source: budowa automatu dopisujacego sekcje "Powiazane" do 460 notatek, 2026-08-01
applies_to: [kazdy skrypt masowo edytujacy pliki w repozytorium]
severity: medium
time_lost: ok. 20 min
---

# Narzedzie masowe, ktore iteruje po zbiorze, przepisuje te same pliki w kolko

## Objaw

Skrypt dopisujacy do notatek liste powiazan zglaszal przy kolejnych uruchomieniach:
46 plikow zmienionych, potem 51, potem znowu 46. Nic w danych sie nie zmienialo.
Powinien byl zglosic zero, bo raz juz wykonal swoja prace.

W repozytorium objawia sie to jako **codzienny commit z dziesiatkami plikow,
w ktorych nic nie znaczy sie zmienilo** - tylko kolejnosc linii w wygenerowanym bloku.

## Przyczyna

Punktacja kandydatow byla liczona w petli po **zbiorze** tagow:

    for t in moje_tagi:          # moje_tagi to set
        for inny in wg_tagu[t]:
            punkty[inny] += waga[t]

Kolejnosc iteracji po zbiorze napisow w Pythonie **zmienia sie miedzy
uruchomieniami procesu**, bo skrot napisu jest losowany na starcie interpretera.
Przy remisie punktowym wygrywal raz jeden kandydat, raz drugi, i wynik tanczyl.

Sortowanie `key=lambda x: -x[1]` tego nie ratuje: sortowanie jest stabilne,
wiec **utrwala** przypadkowa kolejnosc wejscia zamiast ja usunac.

## Naprawa

Trzy miejsca, wszystkie konieczne:

1. Iteruj po `sorted(zbior)`, nie po zbiorze.
2. Do klucza sortowania dopisz **drugi klucz rozstrzygajacy remisy**, np. nazwe:
   `wynik.sort(key=lambda x: (-round(x[1], 6), x[0]))`.
3. Zaokraglij liczbe zmiennoprzecinkowa w kluczu. Dwie sumy tych samych skladnikow
   dodanych w innej kolejnosci roznia sie na ostatnim bicie i tworza sztuczny remis,
   ktory raz jest remisem, a raz nie.

## Bramka, ktora to lapie

Uruchom narzedzie **trzy razy pod rzad**. Drugi i trzeci przebieg musza zglosic zero zmian.

    for i in 1 2 3; do python narzedzie.py; done

To jest tani sprawdzian i wylapuje cala rodzine bledow: nie tylko kolejnosc iteracji,
ale tez znaczniki czasu wpisywane do generowanej tresci i sortowanie po polu,
ktore sie zmienia.

## Dlaczego to wazne poza estetyka

Narzedzie, ktore przy kazdym przebiegu dotyka pol repozytorium, **zabija przydatnosc
historii zmian**. Nie da sie zobaczyc, co naprawde sie zmienilo, bo prawdziwa zmiana
tonie w szumie. Po tygodniu nikt nie patrzy na te commity, a wtedy przestaje dzialac
jedyny mechanizm, ktory pokazywalby, ze automat zwariowal.
