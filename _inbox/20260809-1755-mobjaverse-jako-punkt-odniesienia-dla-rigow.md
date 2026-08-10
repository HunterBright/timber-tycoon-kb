---
title: Mobjaverse - nazwy kosci sa puste w 80 procentach, wiec to punkt odniesienia dla struktury, nie zrodlo animacji
type: lesson
status: draft
confidence: high
verified: '2026-08-09'
date: 2026-08-09
project: GameDevOS
tags:
- mobjaverse
- rigowanie
- czworonogi
- zbiory-danych
- retargeting
- unity
applies_to:
- Unity 6000.5
- Blender 5.2 LTS
source: 'pomiar wlasny na 60 czworonogach ze zbioru, 2026-08-09'
severity: medium
suggested-category: engine/lessons
time_lost: ''
---

# Mobjaverse: struktura tak, nazwy nie

## Problem

Zbior [Mobjaverse](https://huggingface.co/datasets/duckduckplz/Mobjaverse)
(18 914 zrigowanych stworzen, w tym 1497 czworonogow) zostal opisany jako
material do zaprojektowania wlasnego rigu potwora i punkt odniesienia do oceny
auto-riggerow. Opis byl trafny, ale **pominal jedna wlasciwosc, ktora zmienia
sposob uzycia**.

Pomiar na probie 60 czworonogow (`datalist/train/quadruped_128.txt`, pierwsze 60):

**W 48 z 60 modeli pole `joint_names` zawiera dokladnie jedna wartosc: `none`.**
Kosci nie maja nazw.

W pozostalych 12 nazwy sa, ale **w szesciu roznych konwencjach naraz**:

| konwencja | przyklad |
|---|---|
| Maya | `Spine1SkinJnt`, `LeftHipSkinJnt` |
| 3ds Max Biped | `Bip001 Pelvis`, `Bip001 L Thigh` |
| Mixamo | `mixamorig:Hips_01`, `mixamorig:Spine_02` |
| wlasna z prefiksem gatunku | `hyena_Spine_01SHJnt_02` |
| wlasna opisowa | `Waist`, `Neck`, `Head`, `LEar`, `Jaw` |
| bez znaczenia | `base_01`, `Joint_3_03`, `Joint_4_04` |

Oznaczenie strony L/P wystepuje tylko w **3 z 12** modeli z nazwami.

## Root cause

Zbior jest pochodna Objaverse-XL, gdzie modele pochodza od tysiecy roznych
autorow, z roznych programow i potokow. Ujednolicono **geometrie i format**,
a nie **nazewnictwo**. Nazwy przetrwaly tylko tam, gdzie byly w oryginale
i przezyly konwersje.

## Solution

**Czego z tego zbioru NIE da sie zrobic:** przeniesienia animacji po nazwach
kosci. Kazde narzedzie dopasowujace szkielety po nazwach - w tym Animal Animator,
ktory dziala na nazwach kosci metarigu Rigify - **nie ma tu z czym sie zwiazac
w 80 procentach przypadkow**. To nie jest brak, ktory da sie nadrobic skryptem,
bo w 48 modelach nie ma zadnej informacji do odtworzenia.

**Co da sie zrobic, i to dobrze:** czytac sama STRUKTURE. Wyniki z 53 modeli,
w ktorych wykrycie osi symetrii przekroczylo 50% pewnosci (srednia jakosc 97%):

| miara | wynik |
|---|---|
| liczba kosci | zakres 9-117, **mediana 30** |
| glebokosc hierarchii | zakres 3-21, mediana 7 |
| dlugosc lancucha nogi | **3-4 kosci przewazaja, mediana 4** |
| dlugosc lancucha osiowego (ogon/szyja) | mediana 4 |
| rozgalezien na szkielet | mediana 3 |
| symetria lewo-prawo | **43 z 60 modeli powyzej 90%** |
| wierzcholki siatki | mediana 2902 |
| trojkaty | mediana 3944 |

**Najwazniejszy pojedynczy wynik dla Unity:** we **wszystkich 60 modelach**
maksymalna liczba kosci wplywajacych na jeden wierzcholek wynosi **4**, a srednia
1,97. Czyli caly zbior miesci sie w domyslnym limicie Unity (4 wplywy na
wierzcholek) **bez zadnej konwersji ani utraty jakosci**. To jest liczba, ktora
warto znac przed ustawianiem `Skin Weights` przy imporcie.

## What didn't work

**Wykrywanie osi pionowej przez najmniejsza rozpietosc.** Dla zwierzecia najwezsza
os to **lewo-prawo**, nie pion, wiec heurystyka klasyfikowala nogi jako ogony.
Poprawka: os lewo-prawo wykrywa sie **testem symetrii lustrzanej** (odbic
polozenia wzgledem srodka osi i policzyc, ile kosci ma bliskiego sasiada), a pion
odroznia sie od dlugiej osi po tym, ze konce lancuchow skupiaja sie przy jednym
koncu. Po poprawce jakosc wykrycia osi wzrosla do 97%, a liczba modeli
z czterema wykrytymi nogami z 6/58 na 14/53.

**Zastrzezenie, ktore zostaje:** wykrywanie samej LICZBY nog dalej jest slabe
(14 z 53), bo nogi dzielace punkt rozgalezienia zlewaja sie w jeden lancuch.
Dlugosci lancuchow sa wiarygodne, liczba nog nie - i tak zostalo to opisane.

## Transferability

Dwie rzeczy przenosza sie poza ten projekt:

1. **Przed uznaniem zbioru zrigowanych modeli za zrodlo animacji sprawdz, czy
   kosci maja nazwy.** Metadane o liczbie modeli i klas budowy ciala nic o tym
   nie mowia, a to rozstrzyga, czy zbior nadaje sie do przenoszenia ruchu, czy
   tylko do analizy ksztaltu.
2. **Licencja zbioru nie jest licencja modelu.** Tu ODC-BY dotyczy zbioru jako
   calosci, a poszczegolne modele pochodza z Objaverse-XL, gdzie kazdy ma wlasne
   warunki. W plikach **nie ma pola z pochodzeniem ani autorem**, wiec nie da sie
   dojsc, czyj jest konkretny model. Siatki nie moga trafic do produktu i to sie
   nie zmieni.

## Related

- [[20260809-1740-rigify-wolf-nie-jest-rigiem-do-gry]] - mediana 30 kosci stad byla podstawa
  do odrzucenia metarigu wilka (197 kosci deformujacych) na rzecz
  `basic_quadruped`

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260809-1740-rigify-wolf-nie-jest-rigiem-do-gry|Metarig wilka w Rigify to rig filmowy, nie growy - do gry idzie basic_quadruped]] - wspolne: rigowanie, czworonogi
<!-- /POWIAZANE:auto -->
