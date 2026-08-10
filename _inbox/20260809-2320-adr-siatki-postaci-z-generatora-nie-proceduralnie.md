---
title: ADR - siatki postaci bierzemy z generatora, proceduralnie robimy wszystko PO siatce
type: decision
status: draft
confidence: high
verified: '2026-08-09'
date: 2026-08-09
project: GameDevOS
tags:
- postacie
- npc
- proceduralne-modelowanie
- generator
- hunyuan
- loft
- adr
applies_to:
- Blender 5.2 LTS
- Unity 6000.5
source: 'proba wlasna z 09.08.2026, werdykt Huntera'
suggested-category: engine/decisions
---

# ADR: siatki postaci z generatora, proceduralnie tylko to, co PO siatce

## Decyzja (Hunter, 2026-08-09)

**Siatka i tekstura postaci powstaja w Hunyuanie (interfejs webowy).**
Proceduralne modelowanie postaci w Blenderze **odpada jako droga glowna.**

## Kontekst

W jednej sesji zbudowalem od zera dwa modele: goblina (metakule plus remesh)
i klienta w koszulce w stylu zatwierdzonej rodziny NPC z Kerfa (loft z funkcja
profilu, potem jedna ciagla bryla z rzezbieniem po kacie). Referencja byla
jednym zdjeciem: `_BlenderOutputs/NPC_Rodzina/ZATWIERDZONE/06_klient_w_koszulce.png`.

Pomiar poszedl dobrze: proporcje wyciagniete z pikseli co 3% wzrostu, kolory
pobrane z oswietlonych miejsc, rozpietosc ramion odtworzona w granicach 2%.
**Ksztalt nie poszedl.** Werdykt oka: „zle to wyglada, zostaje przy Hunyuanie".

## Dlaczego proceduralne modelowanie postaci przegrywa - trzy powody

1. **Modelowanie przez liczby to praca na slepo.** Ksztalt powstaje z tabel
   wspolrzednych, a wynik widac dopiero po przebudowie i renderze. Czlowiek
   rzezbiacy widzi forme ciagle i koryguje ja tysiace razy; tu jest kilkanascie
   spojrzen na caly model.
2. **Loft nie umie rozgalezien, a cialo sklada sie z rozgalezien.** Bark, biodro
   i nadgarstek to wlasnie te miejsca. Zostaja dwie zle drogi: jedna bryla bez
   konczyn albo konczyny doklejone osobno (zlepek). **Zla jest reprezentacja,
   nie parametry.** Dla smoka loft dziala, bo smok to jedna dluga rura plus
   skrzydla. Humanoid nie jest rura.
3. **Generator nie liczy geometrii, tylko widzial miliony postaci.** Nie
   wyprowadza walu brwiowego z katow i amplitud - zna ksztalt. Reczne
   wyprowadzanie anatomii we wspolrzednych to najdluzsza droga do tego celu.

## Co zostaje po naszej stronie i jest realna przewaga

- **pomiar referencji z pikseli**: proporcje i kolory zamiast szacowania
- **szkielet dopasowany do siatki**, bo znamy wspolrzedne stawow albo umiemy je
  zmierzyc; komplet 15 kosci wymaganych przez Unity Humanoid
- **skorowanie z limitem 4 wplywow** na wierzcholek, zero wierzcholkow bez wag
- **czyszczenie i kontrola siatki z generatora** (dlug geometryczny, winding,
  zerowa grubosc, osie kosci)
- **warianty i eksport**, bramki jakosci, powtarzalnosc

## Konsekwencje

- Nie inwestujemy juz czasu w proceduralne siatki postaci i potworow.
- **Proceduralne modelowanie zostaje tam, gdzie sie sprawdzilo**: propy,
  pojazdy, teren, roslinnosc, ksztalty geometryczne - czyli rzeczy bez
  rozgalezien organicznych.
- Do generatora: Hunyuan przez interfejs webowy. API Tencenta w regionie
  europejskim bylo 09.08 nieprzejezdne (patrz RYTM-STAN), kredyty czekaja.
- Animacje: **Kimodo lokalnie, nie Mixamo** - decyzja Huntera z 09.08.

## Lekcja przenosna

**Zanim zbudujesz narzedzie do generowania ksztaltu, sprawdz, czy wybrana
reprezentacja w ogole potrafi wyrazic docelowa forme.** Loft wyraza rure.
Metakule wyrazaja zlana mase. Zadne z nich nie wyraza czlowieka. Zadna liczba
iteracji nad parametrami tego nie zmieni, a kazda iteracja wyglada na postep.

## Related

- [[20260809-2140-metakule-i-remesh-to-technika-bazowa-nie-wykonczeniowa]]
- [[20260801-0700-adr-npc-od-zera-zamiast-solvera-warstw]] - dwie wczesniejsze
  proby NPC, ta sama lekcja o architekturze
- [[20260720-0915-loft-nie-da-plaskiej-szyby]] - granice loftu, przy autach

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260809-2140-metakule-i-remesh-to-technika-bazowa-nie-wykonczeniowa|Metakule plus remesh wokselowy to technika BRYLY BAZOWEJ, nie wykonczeniowa - do postaci uzywaj loftu z funkcja ksztaltujaca]] - wspolne: loft, proceduralne-modelowanie, postacie
- [[20260801-0700-adr-npc-od-zera-zamiast-solvera-warstw|20260801-0700-adr-npc-od-zera-zamiast-solvera-warstw]] - wspolne: adr, postacie, npc
- [[20260720-0915-loft-nie-da-plaskiej-szyby|Malowanie szyby kolorem na powierzchni z loftu nigdy nie da płaskiej tafli]] - wspolne: loft, proceduralne-modelowanie
<!-- /POWIAZANE:auto -->
