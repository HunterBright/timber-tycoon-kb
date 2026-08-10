---
title: Liczbe postaci w kadrze trzeba powiedziec TWIERDZACO - negatyw jej nie pilnuje
type: lesson
status: draft
confidence: high
verified: 2026-08-01
tags: [generator-obrazow, referencja, prompt, qwen, spojnosc-stylu, npc]
date: 2026-08-01
project: Kerf - Sawmill Tycoon
source: 'generowanie rodziny referencji NPC (robotnik, kasjer, 6 klientow), Qwen-Image, 2026-08-01'
applies_to: [kazde generowanie referencji postaci opisem]
severity: medium
time_lost: ok. 15 min i jeden przebieg
---

# Liczbe postaci w kadrze trzeba powiedziec TWIERDZACO

## Co sie stalo

Generujac osiem referencji NPC w jednym stylu, w opisie negatywnym mielismy
`multiple characters`. Mimo to jedna z osmiu postaci wyszla jako **tlum
identycznych kobiet** - jedna z przodu, siedem w tle.

Negatyw mowil "nie wiele postaci" i model to zignorowal. Dopiero **twierdzenie
w opisie glownym** zadzialalo:

    EXACTLY ONE single character alone in the frame, no other people anywhere.
    (...)
    One person only.

Zdanie postawione **dwa razy**, na poczatku i na koncu bloku kadrowania.

## Zasada

> Negatyw jest slabszym narzedziem niz twierdzenie. Rzeczy policzalne
> (ile postaci, ile obiektow, ile widokow) opisuj **twierdzaco w opisie glownym**,
> a negatyw traktuj jako drugie zabezpieczenie, nie pierwsze.

Ta sama pulapka co przy proporcjach: `chibi` w negatywie nie dawalo smuklej
sylwetki, dopiero jawny opis liczbowy ("siedem i pol glowy") ruszyl wynik.

## Druga rzecz z tego samego przebiegu: negatyw dla RODZINY postaci

Przy pojedynczej postaci negatyw moze byc dowolnie szczegolowy. Przy **rodzinie
postaci w jednym stylu** czesc negatywu zaczyna sobie przeczyc: `beard, moustache`
jest potrzebne, zeby robotnik byl gladko ogolony, ale rownoczesnie starszy klient
MA miec wasy, a barczysty brode.

Rozwiazanie: **wspolny blok negatywu zostaje wspolny i nietkniety**, a kazda postac
dostaje krotka wlasna liste wyjatkow doklejana na koncu. Inaczej albo tracisz
kontrole nad zarostem, albo rozbijasz wspolny blok i postacie przestaja byc rodzina.

Bez tego: robotnik dostal brode mimo slowa `clean-shaven` w opisie.

## Szerszy wniosek o spojnosci stylu

Spojnosc miedzy postaciami bierze sie z tego, co w opisie **zostaje identyczne
co do znaku**, a nie z tego, jak dobrze opisano styl. Uklad, ktory zadzialal:

| Blok | Status |
|---|---|
| styl, proporcje, jezyk twarzy, poza, kadr, negatyw | **wspolne, nietykalne** |
| postac (plec, wiek, budowa, wlosy, skora, ubranie) | **jedyne, co sie zmienia** |
| wyjatki negatywu (np. zarost) | krotka lista per postac |

Osiem postaci zrobionych w ten sposob czyta sie jako jedna rodzina mimo roznego
wieku, plci, budowy i koloru skory.

## Powiazane

- [[20260725-1830-image-model-cannot-force-figure-proportions]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-1600-generator-obrazow-nie-utrzyma-pozy-ani-anatomii-dloni|Generator obrazow nie utrzyma pozy ani anatomii dloni - to nie jest zadanie dla referencji]] - wspolne: generator-obrazow, qwen, referencja
- [[20260809-1145-kimodo-prompt-kropka-dzieli-ruch|20260809-1145-kimodo-prompt-kropka-dzieli-ruch]] - wspolne: prompt, npc
<!-- /POWIAZANE:auto -->
