---
title: Dzwignia --zepsuj musi nazwac winowajce, nie tylko podniesc kod wyjscia
type: pattern
status: draft
confidence: high
verified: ''
date: 2026-08-14
project: Another Quest
tags:
- red-proof
- bramki
- narzedzia-stanu
- python
applies_to: []
source: 'AQ noc D2, job [D2-tools]: tools/findings.py + tools/zbierz_dowody.py'
suggested-category: process/patterns
---

# Dzwignia --zepsuj musi nazwac winowajce, nie tylko podniesc kod wyjscia

## Kontekst
Konwencja narzedzi stanu (skarbiec GameDevOS, AQ tools/): kazde narzedzie ma
`--sprawdz-sie-samemu` i dzwignie `--zepsuj`, ktora wylacza glowne rozroznienie.
Samosprawdzenie z dzwignia MUSI zwrocic PORAZKE - inaczej bramka nie mierzy niczego.

## Wzorzec
Naiwna implementacja pisze proby tak, ze przy `--zepsuj` zadna asercja sie nie
odpala (bo warunki sa napisane pod poprawne dzialanie), a PORAZKE ratuje dopiero
domykajacy warunek "kod zepsuty, a samosprawdzenie przeszlo". Wynik jest formalnie
poprawny (kod 1), ale komunikat nie mowi, CO przeszlo bez zatrzymania. Sedzia
czytajacy raport dostaje "cos jest nie tak" zamiast nazwiska ofiary.

Poprawka jest tania: kazda proba, ktora ma galaz `if zepsuty:`, dostaje `else`
z jawnym komunikatem nazywajacym konkretny wpis/artefakt, ktory przeszedl.

    if zepsuty:
        if kod == 2:
            problemy.append("dzwignia nie zadzialala (regula nadal trzyma)")
        else:
            problemy.append("z dzwignia --zepsuj: 'SYM-90' zostal ZAMKNIETY "
                            "bez zadnego uzasadnienia i narzedzie to przyjelo")

## Dlaczego to wazne
Reguła "bramka bez udowodnionego trybu porazki niczego nie pilnuje" ma dwie
polowki: FAIL **i** nazwanie winowajcy. Domykajacy catch-all daje tylko pierwsza.
Wynik red-proofu trafia do commit message i do paczki dowodowej - jesli jest
ogolnikowy, nastepna osoba nie ma jak sprawdzic, ze proba mierzyla to, co trzeba.

## Zastosowanie
Kazde narzedzie z para `--sprawdz-sie-samemu` / `--zepsuj`. Test kontrolny: uruchom
z dzwignia i przeczytaj wyjscie - jesli nie ma w nim konkretnego identyfikatora
(findingu, pliku, modulu), proba jest niedokonczona.
