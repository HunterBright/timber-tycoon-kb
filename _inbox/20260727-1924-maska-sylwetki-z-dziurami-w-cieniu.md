---
title: Maska sylwetki może mieć dziury w środku — i przez lata tego nie widać
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-27'
project: Kerf - Sawmill Tycoon
tags:
- maska
- sylwetka
- progowanie
- obraz
- referencje
- blender
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Maska sylwetki może mieć dziury w środku — i przez lata tego nie widać

## Objaw

Wycinanie bryły z przecięcia sześciu obrysów dawało prawie pustą bryłę.
Kamery były poprawne (potwierdzone niezależnie), obrysy wyglądały poprawnie,
a mimo to punkt w środku klatki piersiowej wypadał POZA sylwetką na dwóch
z sześciu ujęć.

## Przyczyna

Maska sylwetki powstawała przez progowanie jasności z dodatkową regułą:
„zostaw tylko te piksele, które mają w swojej kolumnie ściankę wyraźnie jasną"
(powyżej progu 120). Reguła miała odsiać cienką siatkę podłogi.

Na ujęciach OD TYŁU model jest zacieniony — środek pleców ma jasność 106.
Nie przechodzi progu 120, więc reguła wycina go z maski. Powstają dziury
w ŚRODKU ciała: 14% powierzchni sylwetki w jednym ujęciu, 13% w drugim.

**Dlaczego nikt tego nie zauważył przez cały wcześniejszy czas:** wszystkie
dotychczasowe pomiary czytały OBRYS — pierwszy i ostatni piksel maski
w wierszu. Dziura w środku nie zmienia ani pierwszego, ani ostatniego. Dopiero
rzeźbienie bryły sprawdza KAŻDY punkt z osobna i dziury natychmiast je wywracają.

## Rozwiązanie

Zalać dziury ZAMKNIĘTE wewnątrz maski (klasyczne `binary_fill_holes`).

Ta operacja **z definicji nie może ruszyć obrysu** — dokłada wyłącznie piksele
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
warto robić zawsze, profilaktycznie — kosztuje nic i niczego nie psuje.

Szerszy wniosek: gdy kontrola przechodzi latami, a nowe użycie tych samych
danych natychmiast pada, to zwykle nie nowe użycie jest zepsute, tylko stara
kontrola patrzyła za wąsko.
