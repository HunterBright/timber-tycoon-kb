---
title: Maska sylwetki może mieć dziury w środku - i przez lata tego nie widać
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-27'
project: Kerf - Sawmill Tycoon
tags:
- mask
- sylwetka
- progowanie
- obraz
- referencja
- blender
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Maska sylwetki może mieć dziury w środku - i przez lata tego nie widać

## Objaw

Wycinanie bryły z przecięcia sześciu obrysów dawało prawie pustą bryłę.
Kamery były poprawne (potwierdzone niezależnie), obrysy wyglądały poprawnie,
a mimo to punkt w środku klatki piersiowej wypadał POZA sylwetką na dwóch
z sześciu ujęć.

## Przyczyna

Maska sylwetki powstawała przez progowanie jasności z dodatkową regułą:
„zostaw tylko te piksele, które mają w swojej kolumnie ściankę wyraźnie jasną"
(powyżej progu 120). Reguła miała odsiać cienką siatkę podłogi.

Na ujęciach OD TYŁU model jest zacieniony - środek pleców ma jasność 106.
Nie przechodzi progu 120, więc reguła wycina go z maski. Powstają dziury
w ŚRODKU ciała: 14% powierzchni sylwetki w jednym ujęciu, 13% w drugim.

**Dlaczego nikt tego nie zauważył przez cały wcześniejszy czas:** wszystkie
dotychczasowe pomiary czytały OBRYS - pierwszy i ostatni piksel maski
w wierszu. Dziura w środku nie zmienia ani pierwszego, ani ostatniego. Dopiero
rzeźbienie bryły sprawdza KAŻDY punkt z osobna i dziury natychmiast je wywracają.

## Rozwiązanie

Zalać dziury ZAMKNIĘTE wewnątrz maski (klasyczne `binary_fill_holes`).

Ta operacja **z definicji nie może ruszyć obrysu** - dokłada wyłącznie piksele
otoczone zewsząd maską. Warto to jednak sprawdzić pomiarem, a nie przyjąć
na słowo: porównałem pierwszy i ostatni piksel maski w KAŻDYM wierszu KAŻDEGO
z sześciu zdjęć przed i po. Zero zmian. Dzięki temu wiadomo, że żaden
z wcześniejszych pomiarów nie drgnie.

Uwaga praktyczna: **Blender nie ma scipy**, a maska musi być identyczna
w Blenderze i poza nim. Zalewanie trzeba więc napisać samemu. Wersja po
odcinkach (run-length) plus przeszukiwanie wszerz od brzegu kadru liczy
obraz 1920×1920 w ułamku sekundy i zgadza się ze scipy co do piksela.

## Reguła ogólna

Jeśli maska powstaje z progowania i ma jakąkolwiek regułę „zostaw tylko to,
co ma obok siebie coś jasnego", to **na zacienionych ujęciach zrobią się
dziury**. Dopóki czytasz z maski tylko obrys, nie zauważysz. Zalanie dziur
warto robić zawsze, profilaktycznie - kosztuje nic i niczego nie psuje.

Szerszy wniosek: gdy kontrola przechodzi latami, a nowe użycie tych samych
danych natychmiast pada, to zwykle nie nowe użycie jest zepsute, tylko stara
kontrola patrzyła za wąsko.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260727-2320-sylwetka-nie-rozdziela-czesci-ktore-sie-stykaja|Sylwetka nie rozdziela dwóch rzeczy, które się stykają - i milczy o tym]] - wspolne: sylwetka, obraz, referencja
- [[20260727-2140-linijka-wysokosci-na-zdjeciu-jest-krzywa|"Ile procent kadru, tyle procent wysokości" to nieprawda, gdy obiektyw patrzy z góry]] - wspolne: obraz, referencja
- [[20260727-1921-kamera-z-siatki-podlogi-nie-z-sylwetki|Kamerę odtwarzaj z regularnej struktury sceny, nie z sylwetki modelu]] - wspolne: referencja, blender
- [[20260728-0030-wymiar-ktorego-nie-widzi-zadne-ujecie-dopasuj-do-wszystkich|Wymiar, którego nie widzi żadne ujęcie, mierzy się dopasowaniem do wszystkich naraz]] - wspolne: sylwetka, blender
- [[20260727-2145-sprawdzaj-czytnik-obrazu-renderem-wlasnego-modelu|Czytnik obrazu sprawdzaj renderem własnego modelu, nie obrazkiem, który sam sobie narysowałeś]] - wspolne: obraz, blender
- [[20260725-1830-plaskie-tekstury-z-plam-referencji|Plaskie tekstury dla modelu z generatora 3D: tozsamosc elementu bierz z REFERENCJI, nie z koloru]] - wspolne: referencja, blender
<!-- /POWIAZANE:auto -->
