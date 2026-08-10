---
type: anti-pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/anti-patterns
tags: [blender, unity, normalne, winding, siatka, pipeline, bramki]
date: 2026-08-07
status: draft
---

# Bezwarunkowe uspojnianie normalnych psuje ZDROWE siatki

## Co probowalismy

W rurze auto-riggujacej modele z generatora 3D stalo bezwarunkowe
`bpy.ops.mesh.normals_make_consistent(inside=False)` zaraz po sklejeniu
importowanych bryl. Uzasadnienie w komentarzu brzmialo sensownie: "Blender rysuje
scianke z obu stron, Unity tylko z jednej - scianka odwrocona robi w grze dziure".

## Dlaczego nie dziala

Modele z generatorow (tryb etapowy Tencent/Hunyuan) to **kilkanascie osobnych bryl
sklejonych w jeden obiekt**, czesto niemanifoldowych: paski wbite w buty, jezyk
buta zachodzacy na cholewke, guziki dotykajace kurtki. Na takiej siatce algorytm
"uspojnij normalne" nie ma jednoznacznego rozwiazania - propaguje orientacje przez
przypadkowe styki i potrafi **odwrocic wiekszosc zdrowej siatki** (zmierzone:
6821 z 9301 scianek). Kolejny etap odwracal calosc z powrotem po objetosci ze
znakiem, ale lokalne kieszenie zostawaly na lewej stronie - w grze dziury.

Wszystkie dotychczasowe kontrole byly na to SLEPE:
- objetosc ze znakiem calosci: dodatnia (duze cialo zjada minus malej muszli),
- objetosc per wyspa: dodatnia (wyspa "but" ma wiecej dobrych scianek niz zlych),
- test promieniem z dwoch stron scianki: BVH ignoruje winding - zawsze 0.

## Zasada

**Operacje naprawcze uruchamiaj tylko wtedy, gdy pomiar mowi, ze jest co naprawiac.**
Naprawa uruchomiona "na wszelki wypadek" na danych, ktorych nie umie zinterpretowac,
jest zrodlem wad, a nie ich lekiem. Wersja w kodzie:

```python
if objetosc_calosci < 0 or wysp_na_lewej_stronie > 0:
    normals_make_consistent(inside=False)
else:
    pisz("zrodlo zdrowe - winding NIETKNIETY")
```

## Jak to wykryc (bramka, ktora dziala)

Renderuj model DWA razy z tego samego ujecia: raz normalnie (silnik rysuje tylko
przednie scianki) i raz materialem dwustronnym. Dla zdrowej bryly sylwetki sa
identyczne. Piksel, ktory w pierwszym renderze jest TLEM, a w drugim MODELEM, to
dokladnie dziura widoczna w grze. Liczba takich pikseli = miara wady.

Bramka ma udowodniony tryb porazki: odwroc winding czesci trojkatow w pamieci
i sprawdz, ze licznik skacze (u nas 1176 trojkatow -> 67005 pikseli).

## Dodatkowo: dwustronne materialy postaci

Buty i nogawki z generatorow to OTWARTE RURY - przez szpare miedzy jezykiem
a nogawka widac czern wnetrza. To nie jest wada pipeline'u (zrodlo mialo tych
pikseli WIECEJ niz nasze wyjscie), tylko cecha modeli. Jedna wlasciwosc materialu
(`_Cull = 0`, Render Face: Both) zamyka wszystkie takie szpary raz na zawsze,
takze w przyszlych modelach. Koszt przy 8 postaciach po ~10k trojkatow: pomijalny.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260731-2115-bramka-ktora-istnieje-ale-nie-odpala-sie-dla-wiekszosci-obiektow|Bramka, ktora istnieje, ale nie odpala sie dla wiekszosci obiektow]] - wspolne: bramki, blender
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: pipeline, blender
- [[20260801-1130-quadriflow-kasuje-uv-i-wagi-bez-jednej-flagi|QuadriFlow kasuje UV i wagi szkieletu, dopoki nie wlaczysz jednej flagi]] - wspolne: pipeline, blender
- [[20260807-1155-skrypt-zarzadzany-blender-bezokienkowy|Skrypt zarzadzany - dyscyplina edycji Blendera bezokienkowo]] - wspolne: pipeline, blender
- [[20260731-2200-slepa-dzwignia-debugger-bramek|20260731-2200-slepa-dzwignia-debugger-bramek]] - wspolne: bramki, blender
- [[20260801-0500-gestszy-pomiar-odslania-dlug|20260801-0500-gestszy-pomiar-odslania-dlug]] - wspolne: bramki, blender
<!-- /POWIAZANE:auto -->
