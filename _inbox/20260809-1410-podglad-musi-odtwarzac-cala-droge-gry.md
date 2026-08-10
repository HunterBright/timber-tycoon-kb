---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [unity, animacja, narzedzia-edytorowe, korekty-kosci, bramki, late-update]
date: 2026-08-09
status: draft
---

# Okno do strojenia musi odtwarzać CAŁĄ drogę gry, nie tylko klip

## Objaw

Reżyser dostroił i zaakceptował pozę postaci w oknie edytorowym z suwakami. Wartości
przepisane do gry 1:1 dały **inny wygląd**: postać przekrzywiona o 5-7 stopni zamiast prosto.

## Przyczyna

Okno pokazywało: `klip -> moje korekty`.
Gra wykonywała: `klip -> istniejący komponent korygujący -> moje korekty`.

W projekcie działał już komponent poprawiający pozę po animatorze (przechył tułowia, obrót
stóp). Okno go nie uruchamiało, bo w trybie edycji `LateUpdate` nie leci. Reżyser stroił więc
przechył od zera, a w grze jego wartość **dodawała się** do istniejącej.

## Dwie pułapki, które to ukryły

1. **Kolejność też jest wynikiem.** Istniejący komponent obraca kręgosłup PRZED ramionami
   (od korzenia do końcówek), a okno robiło to PO. Obrót kręgosłupa niesie ramiona ze sobą,
   więc te same liczby w innej kolejności dają inną pozę. Nie wystarczy odjąć wartości -
   trzeba odtworzyć kolejność.
2. **`Awake` nie leci przy `AddComponent`.** Nowy komponent szukał sąsiada w `Awake`.
   Dokładany z kodu (`AddComponent`) i w narzędziach edytorowych nigdy go nie znajdował,
   więc flaga "pomiń swój przechył" nie wsiadała. Bramka pokazywała wtedy identyczny wynik
   dla przebiegu właściwego i dla dźwigni - i to była jedyna widoczna oznaka.

## Reguła

- Narzędzie do strojenia ma wołać **ten sam łańcuch korekt, co gra**, w tej samej kolejności.
  Jeśli komponent gry robi to w `LateUpdate`, wystaw publiczną metodę `Zastosuj()` i wołaj ją
  z narzędzia.
- Zależności między komponentami rozwiązuj **leniwie** (`if (x == null) x = GetComponent<...>()`),
  nie tylko w `Awake` - inaczej ścieżka `AddComponent` cicho traci połączenie.
- Po przepisaniu zaakceptowanych wartości do gry **zmierz obie drogi** i porównaj liczbą.
  Tu różnica wynosiła 4 stopnie i bez pomiaru weszłaby do buildu.

## Dźwignia bramki

Bramka porównująca "drogę gry" z "podglądem" musi mieć tryb, który **celowo** zostawia podwójną
korektę. Jeśli wynik z dźwignią i bez niej jest identyczny, bramka jest ślepa - dokładnie tak
było, zanim wyszła sprawa z `Awake`.

Powiązane: [[20260809-1145-kimodo-prompt-kropka-dzieli-ruch]].
