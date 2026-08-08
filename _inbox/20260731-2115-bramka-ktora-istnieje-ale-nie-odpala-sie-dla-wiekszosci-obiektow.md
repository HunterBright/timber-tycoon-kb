---
title: Bramka, ktora istnieje, ale nie odpala sie dla wiekszosci obiektow
type: anti-pattern
status: draft
confidence: high
verified: 2026-07-31
tags: [bramki, walidacja, pipeline-assetow, blender, generator]
date: 2026-07-31
project: Kerf - Sawmill Tycoon
source: naprawa listew bocznych aut NPC 2026-07-31
applies_to: [dowolny projekt z walidatorami assetow albo danych]
severity: high
time_lost: ok. 30 min (samo znalezisko), blad zyl w projekcie ~11 dni
---

# Bramka, ktora istnieje, ale nie odpala sie dla wiekszosci obiektow

## Antywzorzec
Walidator ma na wejsciu parametr opcjonalny i zaczyna od:

    if spec.side_step_x is None:
        return []          # nie ma czego sprawdzac

Wyglada rozsadnie: "model, ktory nie ma progu, nie musi go zglaszac". W praktyce znaczy to,
ze **bramka pilnuje tylko tych obiektow, ktore SAME sie do niej zglosza** - a zglasza sie ten,
ktory akurat powstal w dniu, gdy walidator pisano.

## Co sie stalo naprawde
Kontrola "prog nie moze zachodzic na kolo" powstala po tym, jak Hunter zobaczyl blad na rendrze
auta GRACZA. Zostala napisana poprawnie, ma dobry komentarz, dziala. Przez 11 dni pilnowala
**jednego modelu z czterech**: tylko auto gracza podawalo `side_step_x`. Trzy auta NPC nie przeszly
jej ani razu - i to wlasnie w jednym z nich siedzial artefakt (koniec listwy wiszacy w otworze
nadkola), ktorego nikt nie widzial.

Raport zbiorczy pokazywal "walidatory: zielone". Nie klamal. Po prostu nie liczyl, dla ILU
obiektow cokolwiek zmierzyl.

## Jak to lapac
1. **Licz pokrycie, nie tylko wynik.** Walidator ma raportowac takze "sprawdzono N z M obiektow".
   Zielone przy N=1, M=4 to sygnal, nie sukces.
2. **Parametr opcjonalny = domyslnie WLACZONY, nie wylaczony.** Jesli da sie policzyc wartosc
   z danych, ktore obiekt i tak ma (tu: rozstaw osi i promien kola), licz ja, zamiast czekac,
   az ktos ja wpisze recznie.
3. **Wczesne `return []` to miejsce, ktore trzeba czytac podejrzliwie.** Kazde takie wyjscie
   to cicha zgoda na brak pomiaru. Warto przy nim zostawic licznik pominiec.
4. Przy nowym wariancie/obiekcie zapytaj wprost: **ktore istniejace bramki go obejma, a ktore nie**.

## Dowod
- `_BlenderOutputs/NPCCar01/car_spec.py` -> `check_side_step_clears_wheels` z wczesnym wyjsciem
  na `side_step_x is None`; parametru nie podawal zaden `spec_sedan/kombi/hatch`, tylko `spec_pickup`.
- Po dodaniu parametru do trzech specyfikacji NPC i dopisaniu drugiej kontroli
  (`check_side_trim_ends_on_body`) budowa sedana i kombi **zatrzymala sie od razu**:
  `zapas -0.0475, min 0.010 - koniec WISI w otworze nadkola`. Czyli blad byl tam caly czas.
- Progi w autach NPC okazaly sie przy okazji poprawne (0,155 m odstepu przy wymaganych 0,10) -
  szukany blad byl gdzie indziej, niz brzmialo zgloszenie.

## Czy to przeniesie sie na inny projekt
Tak, i nie dotyczy tylko assetow. Ten sam ksztalt ma kazdy walidator danych z opcjonalnym polem,
kazdy lint z `# noqa`, kazdy test parametryzowany lista, ktora ktos zapomnial uzupelnic.

## Powiazane
- [[gate-must-have-provable-failure-mode]]
- [[20260731-1700-trzymanie-boczne-liczone-z-osi-nadwozia-zjada-grawitacje]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260807-0900-normalne-nie-uspojniac-bezwarunkowo|20260807-0900-normalne-nie-uspojniac-bezwarunkowo]] - wspolne: bramki, blender
- [[20260731-2200-slepa-dzwignia-debugger-bramek|20260731-2200-slepa-dzwignia-debugger-bramek]] - wspolne: bramki, blender
- [[20260801-0500-gestszy-pomiar-odslania-dlug|20260801-0500-gestszy-pomiar-odslania-dlug]] - wspolne: bramki, blender
- [[20260807-1620-skinning-lerp-zapada-nadgarstek|20260807-1620-skinning-lerp-zapada-nadgarstek]] - wspolne: bramki, blender
<!-- /POWIAZANE:auto -->
