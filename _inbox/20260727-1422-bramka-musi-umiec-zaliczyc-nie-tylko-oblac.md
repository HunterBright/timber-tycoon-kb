---
title: Bramka musi mieć udowodniony tryb ZALICZENIA, nie tylko PORAŻKI
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-27'
project: Kerf - Sawmill Tycoon
tags:
- testy
- bramki
- kontrola-jakosci
- referencja
- generatory-proceduralne
applies_to: []
source: ''
suggested-category: workflow/lessons
---

# Bramka musi mieć udowodniony tryb ZALICZENIA, nie tylko PORAŻKI

## Objaw

Generator postaci w Blenderze miał komplet kontroli: szczelność powłoki,
symetria położeń co do bitu, symetria układu ścianek, liczba trójkątów,
stałość topologii. Wszystkie świeciły na zielono przez cztery rundy pracy.
Hunter obejrzał wynik i powiedział: *"gdzieś po drodze zgubiłeś swoją
referencję, nogi wyglądają okropnie"*. Miał rację, a żadna kontrola tego
nie widziała.

## Przyczyna

**Każda z tych kontroli sprawdzała, czy siatka zgadza się SAMA ZE SOBĄ.
Ani jedna nie sprawdzała, czy wynik przypomina to, z czego miał powstać.**

Referencja została odczytana RAZ, do ośmiowierszowej tabelki. Wszystko
później liczyło się ze współczynników dobieranych na oko. Dryf kumulował
się cicho, bo nie było miary, która by go zobaczyła.

## Wniosek 1 - miara wierności musi być osobną bramką

Do zestawu kontroli spójności trzeba dołożyć kontrolę PODOBIEŃSTWA do
źródła. Tu: model cięty płaszczyznami co 1% wzrostu, z każdego przekroju
liczona szerokość i głębokość, porównywane z sylwetką wyciętą z pikseli
referencji. Kontrole spójności mówią "nie rozpadło się". Dopiero kontrola
wierności mówi "to jest to, co miało być".

## Wniosek 2 - i to jest ta droższa połowa lekcji

Bramka, która potrafi tylko OBLAĆ, jest podejrzana. Trzeba udowodnić oba
tryby:

1. **Tryb porażki**: puścić bramkę na znanym złym wyniku i sprawdzić, że
   wskazuje palcem to samo, co człowiek zobaczył gołym okiem.
2. **Tryb zaliczenia**: puścić ją na ATRAPIE zbudowanej wprost z odczytanych
   liczb - tu były to rury złożone ze zmierzonych średnic. Jeśli bramka
   oblewa nawet taką atrapę, to cel jest nieosiągalny i zepsuta jest
   bramka, nie model.

Ten drugi dowód od razu się zwrócił: atrapa wyszła dokładnie 1,75 raza za
chuda, bo promienie były w ułamkach wzrostu, a wysokości w metrach. Bez
niego szedłbym za celem, którego nie da się trafić, i "poprawiał" model.

## Trzecia rzecz: porównuj to samo z tym samym

Referencja to płaskie zdjęcie - głowa i bark na jednej wysokości dają jedną
plamę, bo jedno zasłania drugie. Model to bryła i daje dwa osobne przekroje.
Zanim cokolwiek porównasz, przekroje modelu trzeba skleić dokładnie tak, jak
skleja je rzutowanie na zdjęcie. Inaczej różnica w wyniku mówi o różnicy
w licznikach, a nie o różnicy między modelem a wzorcem.

## Kiedy to stosować

Wszędzie, gdzie coś jest generowane kodem "na podstawie" wzorca: postacie,
teren z mapy wysokości, UI z projektu graficznego, dane z dokumentu.
Wszędzie tam zestaw testów naturalnie ciąży ku sprawdzaniu spójności wyniku,
bo to jest łatwe do napisania - a milczy o tym, czy wynik ma cokolwiek
wspólnego ze źródłem.

## Powiązane

- [[feedback_probe_must_be_able_to_fail]] - starsza połowa tej samej zasady:
  każda kontrola musi mieć udowodniony tryb porażki. Ta lekcja dokłada drugą
  połowę: musi mieć też udowodniony tryb zaliczenia.
