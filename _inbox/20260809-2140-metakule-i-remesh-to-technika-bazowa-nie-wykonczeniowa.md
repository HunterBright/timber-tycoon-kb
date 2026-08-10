---
title: Metakule plus remesh wokselowy to technika BRYLY BAZOWEJ, nie wykonczeniowa - do postaci uzywaj loftu z funkcja ksztaltujaca
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-09'
date: 2026-08-09
project: GameDevOS
tags:
- blender
- proceduralne-modelowanie
- low-poly
- postacie
- potwory
- metakule
- remesh
- loft
applies_to:
- Blender 5.2 LTS
source: 'pomiar wlasny plus analiza cudzego wyniku, 2026-08-09'
severity: high
suggested-category: engine/anti-patterns
time_lost: 'cztery iteracje modelu goblina w jednej sesji'
---

# Metakule plus remesh to technika bryly bazowej, nie wykonczeniowa

## Co probowalismy zrobic

Zbudowac proceduralnie goblina low poly do gry: bryla z metakul (dodatnich na mase
ciala, ujemnych na oczodoly i usta), potem remesh wokselowy dla ujednolicenia,
potem decymacja do budzetu, UV, wypalona tekstura i szkielet.

Wszystkie kroki techniczne wyszly: 22 kosci, komplet 15 kosci wymaganych przez
Unity Humanoid, maks 4 wplywy na wierzcholek, zero wierzcholkow bez wag, cztery
warianty z jednego skryptu. **Ale ksztalt byl mydlany i nie przeszedl oka rezysera.**

## Dlaczego to nie dziala

Dwa mechanizmy niszcza detal, kazdy osobno wystarczy:

1. **Metakule z definicji zlewaja wszystko w jedna gladka mase.** Nie da sie nimi
   zrobic polki brwiowej, nawisu wargi ani kilu grzbietowego, bo one wygladzaja
   dokladnie to, co mialoby byc krawedzia. Metakule ujemne wycinaja wglebienia,
   ale tez gladko.
2. **Remesh wokselowy probkuje ksztalt na siatce o staly krok** (u nas 0,013 m)
   i wszystko cienszego albo ostrzejszego po prostu znika.

Objaw, ktory to zdemaskowal: przy skalowaniu ucha do 1,45 ucho robilo sie dluzsze,
ale promien zostawal ten sam, wiec schodzilo ponizej rozmiaru woksela, **rozpadalo
sie na kawalki, a czyszczenie „zostaw najwieksza bryle" kasowalo je bez slowa**.
Model wygladal jak goblin bez uszu i nic nie zglaszalo bledu.

## Co robic zamiast tego

**Loft z analityczna funkcja ksztaltujaca**, czyli technika, ktora daje wynik klasy
pokazowej (zweryfikowane na cudzym, opublikowanym przykladzie smoka):

- jeden ciagly lancuch przekrojow wzdluz krzywej kregoslupa (rzad 100-330 pierscieni
  po 32-72 punkty), zszyty w siatke o **regularnej, czworokatnej topologii**
- promien i profil pierscienia jako **funkcja polozenia**, ktora rzezbi NAZWANE
  cechy: dach czaszki, polka brwiowa, nawis wargi, klatka zebrowa, splaszczenie
  brzucha, masy barku i zadu, kil grzbietowy
- **detal jako osobne przenikajace sie bryly**, nie jako faktura jednej powloki:
  zuchwa osobno, uszy osobno, luski jako pasy, kolce jako instancje
- **nie walcz o low poly podczas modelowania** - buduj z pelna definicja, redukuj
  na samym koncu
- kolor w wierzcholkach zamiast wypalania, gdy to tylko plaskie regiony

## Co dziala, a co nie - rozroznienie, ktore latwo zgubic

Nasz wlasny ADR z 01.08 mowi, ze „gesta siatka nie kupuje jakosci w low poly, kupuje
faldy". **To jest prawda o gestej siatce z RZEZBIENIA i solvera**, popychanej przez
mechanizmy dopasowania. Gesta siatka z **loftu analitycznego** jest czym innym -
jest wyliczona, wiec nie ma czego marszczyc. Te dwa zdania nie sa sprzeczne, ale
wygladaja na sprzeczne, jesli sie nie doda zrodla gestosci.

## Powiazanie z tym, co juz wiedzielismy

[[20260720-0915-loft-nie-da-plaskiej-szyby]] konczy sie zaleceniem „rozbic bryle na
dwie przenikajace sie bryly zamkniete". **To jest dokladnie ta sama zasada, tylko
zastosowana do auta, a nie do postaci.** Wiedza byla u nas od 20.07 i nie zostala
przeniesiona na postacie, bo mapa low poly ma w luce wpisane wprost: „Modele potworow
i stworzen - zero wpisow".

## Transferability

Dotyczy kazdego proceduralnego modelowania organicznych ksztaltow w Blenderze,
niezaleznie od silnika i gatunku gry. Regula do zapamietania w jednym zdaniu:
**metakule i remesh sluza do znalezienia bryly, loft z funkcja profilu sluzy do
zbudowania modelu.**

Osobno transferowalne: **czyszczenie typu „zostaw najwieksza bryle" musi glosno
raportowac, gdy usuwa cos duzego** - inaczej kasuje czesci ciala i wyglada to na
ceche, nie na usterke. U nas prog 3% siatki.

## Related

- [[20260720-0915-loft-nie-da-plaskiej-szyby]] - ta sama zasada przy autach
- [[20260801-0700-adr-npc-od-zera-zamiast-solvera-warstw]] - dwie wczesniejsze
  proby NPC i lekcja „walczysz z architektura, nie z bugami"

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260809-2320-adr-siatki-postaci-z-generatora-nie-proceduralnie|ADR - siatki postaci bierzemy z generatora, proceduralnie robimy wszystko PO siatce]] - wspolne: loft, proceduralne-modelowanie, postacie
- [[20260720-0915-loft-nie-da-plaskiej-szyby|Malowanie szyby kolorem na powierzchni z loftu nigdy nie da płaskiej tafli]] - wspolne: loft, proceduralne-modelowanie, low-poly
- [[20260801-0700-adr-npc-od-zera-zamiast-solvera-warstw|20260801-0700-adr-npc-od-zera-zamiast-solvera-warstw]] - wspolne: postacie, low-poly
- [[20260726-1120-zero-thickness-surfaces-break-voxel-remesh|Zero-thickness surfaces make voxel remesh amputate parts of a model]] - wspolne: remesh, blender
- [[20260731-1050-rowne-krawedzie-ubran-bisect-plane|20260731-1050-rowne-krawedzie-ubran-bisect-plane]] - wspolne: low-poly, blender
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: low-poly, blender
<!-- /POWIAZANE:auto -->
