---
title: Okno modalne trzymajace uchwyt do przedmiotu z rak gracza kasuje towar
type: anti-pattern
status: draft
confidence: high
verified: 2026-07-31
tags: [ui, ekwipunek, modal, utrata-danych, unity]
date: 2026-07-31
project: Kerf - Sawmill Tycoon
source: zgloszenie testera 2026-07-27 ("ukradlo mi drzewo z magazynu cale")
applies_to: [dowolna gra z oknem przekladania towaru miedzy pojemnikami]
severity: high
time_lost: ok. 1,5 h
---

# Okno modalne trzymajace uchwyt do przedmiotu z rak gracza kasuje towar

## Antywzorzec
Okno transferu (magazyn <-> pojemnik w rekach) zapamietuje referencje do pojemnika przy
otwarciu i uzywa jej przy kazdym kliknieciu, **nie sprawdzajac po drodze, czy gracz wciaz go
trzyma**. Do tego zawartosc pojemnika jest utrwalana dopiero przy jego chowaniu.

## Dlaczego to nie dziala
Wystarczy JEDNA droga, ktora zabiera przedmiot z rak przy otwartym oknie (u nas kolo narzedzi
pod [Q], niezablokowane nad oknem). Wtedy:

1. Pojemnik znika z rak, ale obiekt dalej istnieje (tylko wylaczony) - referencja NIE jest null,
   wiec zaden `if (x == null)` tego nie zlapie.
2. Kolejne kliki dzialaja "normalnie": towar wychodzi z magazynu i wchodzi do pojemnika-widma.
3. Przy nastepnym wyjeciu pojemnika gra odtwarza go z **zapamietanej migawki** - i nadpisuje
   wszystko, co wpadlo tam po zniknieciu. Towar znika z gry bez sladu w logu.

Klik "maks" (Ctrl) robi z tego jednorazowa utrate calego magazynu.

## Jak robic zamiast tego
Trzy zamki, kazdy tani:

1. **Zrodlo prawdy zamiast referencji.** Jedna metoda "co gracz TERAZ trzyma" i okno pyta o nia
   PRZED kazda operacja na towarze. Rozne miejsca nie moga mieć wlasnych sposobow sprawdzania.
2. **Zamknij droge ucieczki.** Zmiana narzedzia (kolo, skroty) zablokowana nad otwartym oknem
   modalnym - ta sama lista, ktora blokuje interakcje ze swiatem.
3. **Utrwalaj natychmiast.** Migawka zawartosci odswiezana po KAZDEJ operacji, nie przy chowaniu.
   Wtedy zadna nieznana droga wyjscia nie kasuje juz przelozonego towaru.

Zamek 1 sam wystarczy na zgloszony blad; 2 i 3 kosztuja po kilka linijek i zamykaja cala rodzine.

## Pulapka przy naprawie
Twarde "zamykaj okno, gdy gracz nie trzyma pojemnika" w kazdej klatce **zepsulo bramke sondy**,
ktora mierzyla geometrie okna, otwierajac je na atrapie pojemnika. Podzial:
- twardy zamek przy samym przekladaniu towaru (tam siedzi szkoda),
- lagodny przy odswiezaniu okna (zamykaj, gdy pojemnik zginal albo gracz trzyma INNY).

## Dowod
Sonda buildowa Kerf: `UI/Regal nisze` i `UI/Skrzynia nisze` przeszly z PASS na FAIL po zbyt
agresywnej wersji zamka, i wrocily na PASS po podziale. Zielony przebieg `passed 243/243`.

## Czy to przeniesie sie na inny projekt
Tak. Wzorzec "okno modalne + przedmiot w rekach" jest w kazdym tycoonie i kazdym sklepie w grze.
Ogolna zasada: **referencja zapamietana przy otwarciu okna nie jest dowodem, ze stan wciaz trwa**.

## Powiazane
- [[gate-must-have-provable-failure-mode]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260613-0610-dim-scrim-must-not-reuse-9slice-panel-factory|Don't build a full-screen dim/scrim by reusing your skinnable panel factory]] - wspolne: modal, ui
- [[20260617-1210-tmp-text-legibility-on-textured-bg|TextMeshPro: czytelność na teksturowanym tle (drewno) + warstwy modali]] - wspolne: modal, ui
<!-- /POWIAZANE:auto -->
