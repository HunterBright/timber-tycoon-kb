---
title: Blender nie przelicza `image.pixels[]` na sRGB przy zapisie PNG
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-25'
project: Kerf - Sawmill Tycoon
tags:
- blender
- python
- color-management
- srgb
- gamma
- texture
- palette
- unity
applies_to: []
source: ''
severity: high
suggested-category: engine/lessons
---

# Blender nie przelicza `image.pixels[]` na sRGB przy zapisie PNG

## Objaw

Skrypt tworzy w Blenderze male teksture-palete (jednolite kwadraty o zadanych kolorach),
wpisujac wartosci do `image.pixels[]` i zapisujac przez `image.save()`.
W renderze **kolory sa za ciemne i za nasycone** - skora zadana jako `#E8AE80` wygladala
jak `#C86A37`. Przez kilka iteracji winowajca wydawal sie **dobor koloru**, a nie zapis pliku.

## Przyczyna

`image.pixels[]` to bufor **liniowy** (float). Zapis do PNG **nie robi** konwersji
liniowa->sRGB, wiec do pliku ida wartosci liniowe potraktowane jak sRGB. Efekt to
przyciemnienie o krzywa gamma (~2,2). Podglad w Blenderze tego nie ujawnia, bo jego
przetwornik widoku dokladnie te przesuniecie w druga strone kompensuje przy wyswietlaniu.

To ten sam rodzaj pulapki co wczesniejsze: **wypiek danych do obrazka 8-bitowego
przepuszcza je przez krzywa sRGB** - tylko w druga strone.

## Rozwiazanie, ktore dziala

Nie polegac na Blenderze przy zapisie palety. **Wlasny zapis PNG bajt w bajt**
(zwykly zlib + CRC, kilkadziesiat linii) z wartosciami sRGB dokladnie takimi,
jakie maja byc w pliku.

Alternatywy, ktore trzeba sprawdzic ZA KAZDYM RAZEM, jesli sie na nie zdecydujesz:
ustawienie `image.colorspace_settings.name` oraz przeliczenie liniowa->sRGB
przed wpisaniem do bufora. Nie zakladac, ze zadzialalo - **zmierzyc**.

## Jak to pilnowac

Kontrola musi czytac **BAJTY PLIKU**, nie bufor w pamieci. Kontrola czytajaca
`image.pixels[]` przechodzi zawsze i jest bezwartosciowa - bufor jest przeciez zgodny
z tym, co wpisal skrypt. Dopiero porownanie bajtow pliku z zadana tablica kolorow
wykrywa rozjazd.

Druga kontrola, warta zachodu: **zmierzyc kolor w renderze** i porownac z paleta.
Po naprawie najjasniejszy policzek mierzyl `#E5AD81` przy palecie `#E8AE80` - to potwierdza,
ze i paleta, i kalibracja swiatla studyjnego sa dobre. Bez tego pomiaru nie wiadomo,
czy patrzy sie na blad tekstury, czy na blad oswietlenia scenki.

## Zasieg

Dotyczy **kazdej** tekstury generowanej skryptem w Blenderze, nie tylko palet postaci.
W tym projekcie kolejne etapy (koszula, kamizelka, spodnie, fartuch sprzedawcy) uzywaja
tej samej palety - gdyby ktos wygenerowal ja "po staremu", Unity dostaloby przyciemnione
kolory calej postaci.

Patrz tez: [[feedback_probe_must_be_able_to_fail]],
[[project_worker_texture_painted_2026-07-25]] (pulapka nr 2: wypiek do 8-bit psuje wspolrzedne).
