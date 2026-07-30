---
type: pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/patterns
tags: [pomiar, kamery, fotogrametria, sylwetka, dopasowanie, blender]
date: 2026-07-28
status: draft
---

# Wymiar, którego nie widzi żadne ujęcie, mierzy się dopasowaniem do wszystkich naraz

## Problem

Ręka postaci wisi wzdłuż ciała. Z przodu widać ją osobno, więc szerokość
i odległość od osi są odczytane wprost. Ale **z boku ręka leży dokładnie na
tle tułowia** i sylwetka ich nie rozdziela, więc grubości (wymiaru w głąb)
nie widać w żadnym pojedynczym rzucie.

Dwa oczywiste podejścia zawiodły:

1. **Przecięcie sześciu obrysów (visual hull)** rozdzieliło rękę od tułowia
   wzorowo, a jej szerokość zgadzała się z sylwetką co do półcentymetra.
   Ale głębokość wyszła 18 cm przy szerokości 7,6 cm, bo żadne z sześciu ujęć
   nie tnie tego wymiaru - rzeźbienie zostawia długi pryzmat.
2. **Czytanie narysowanych krawędzi** w rzucie z boku (obrys ręki jest tam
   narysowany na tle tułowia). Tors ma w tym miejscu tyle własnych pionowych
   krawędzi, że histogram wychodzi płaski.

## Wzorzec

Nie szukaj ujęcia, które widzi szukany wymiar. Zamiast tego **weź wszystkie
ujęcia, w których obiekt jest osobną plamą, i dopasuj model tak, żeby zgadzał
się ze wszystkimi naraz.**

U nas: ręka stoi jako osobna grupa sylwetki w czterech ujęciach (z tyłu
i trzy skośne). Kamery były wcześniej rozwiązane, więc dla każdej dało się
policzyć, gdzie i jak szeroko wypadnie w kadrze walec o zadanym środku
i zadanej grubości:

    kolumna  = środek rzutowany kamerą
    szerokość = 2 * f/z * sqrt(rx^2 cos^2(azymut) + ry^2 sin^2(azymut))

Dwie niewiadome (jak głęboko wisi, jak jest gruba) wobec ośmiu obserwacji
(cztery ujęcia razy środek i szerokość). Przewymiarowane, więc ma z czego
oblać. Rezyduum zeszło do 6,8 piksela.

## Jak sprawdzić, że to naprawdę mierzy

Samo małe rezyduum nic nie dowodzi - trzeba pokazać, że **zły wynik pasuje
wyraźnie gorzej**. Dwa testy:

- ręka dwa razy grubsza: błąd rośnie 3,1 raza,
- ręka przesunięta o 8 cm w głąb: błąd rośnie 8,3 raza.

Gdyby dopasowanie nic nie mówiło o grubości, oba stosunki wyszłyby ~1,0.
Warto zapisać próg wprost i uzasadnić jego wysokość: głębokość jest tu
najsłabszą z czterech liczb, bo wychodzi tylko pośrednio, ze skosów.

## Pułapka, która psuje wszystko po cichu

Wiersz obrazu, w którym szuka się obiektu, **musi pochodzić z rzutowania
punktu 3D przez tę kamerę**, a nie z proporcji „ile procent wysokości kadru".
Przy kamerze patrzącej z góry proporcja jest krzywa: u nas ujęcie z pochyleniem
38 stopni myliło się o 11% wysokości postaci, czyli 20 cm. „Ta sama wysokość"
w dwóch ujęciach wypadała w zupełnie innych miejscach ręki i dopasowanie
skakało bez ładu.

Objaw tej pomyłki jest charakterystyczny: wynik nie jest losowy, tylko
**niestabilny między sąsiednimi przekrojami** - dwie sąsiednie wysokości
dają wyniki różniące się dwukrotnie.
