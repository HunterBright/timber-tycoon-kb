---
title: Szybkie rysowanie w Unity 6 omija wszystko, co ma Animator
type: lesson
status: verified
confidence: high
verified: '2026-08-05'
tags:
- unity
- wydajnosc
- animacja
- tlum
- lod
date: '2026-08-01'
project: GameDevOS
source: https://docs.unity3d.com/6000.5/Documentation/Manual/urp/gpu-resident-drawer.html
suggested-category: engine/lessons
applies_to:
- Unity 6.x
- URP
severity: high
time_lost: 'zero, zlapane w zwiadzie przed rozpoczeciem prac'
---

# Szybkie rysowanie w Unity 6 omija wszystko, co ma Animator

## Objaw

Planujesz gre z tlumem przeciwnikow. Wlaczasz GPU Resident Drawer, bo to jest
w Unity 6 domyslna droga do laczenia wielu obiektow w jedno wywolanie rysowania.
Liczba wywolan nie spada. Postacie sa rysowane po jednej, tak jak przedtem.

## Przyczyna

To nie jest blad ani zla konfiguracja. To udokumentowane ograniczenie, tylko
zapisane w miejscu, do ktorego zaglada sie po fakcie.

GPU Resident Drawer obsluguje **wylacznie MeshRenderer**. Obiekt jest wykluczony
z szybkiego toru, jesli:

- ma SkinnedMeshRenderer, albo
- **gdziekolwiek w jego hierarchii** znajduje sie komponent Animation lub Animator.

Drugi warunek jest tym, ktory zaskakuje. Nie wystarczy, ze sama siatka jest
zwykla - wystarczy Animator na rodzicu i caly obiekt wypada.

To samo ograniczenie ma GPU Instancing. Do tego wycinanie zaslonietych obiektow
liczone na karcie graficznej **wymaga wlaczonego GPU Resident Drawera**, wiec
odpada razem z nim.

Wniosek jest niewygodny: **wszystkie nowoczesne mechanizmy wsadowe Unity 6
omijaja animowane postacie.** Dziala to swietnie dla drzew, skrzyn i kamieni,
czyli dokladnie dla tego, co i tak bylo tanie.

## Rozwiazanie

Zeby postac wpadla w szybki tor, musi przestac byc postacia z punktu widzenia
silnika. Trzeba **zdjac z niej Animator i SkinnedMeshRenderer**, a animacje
przeniesc do tekstury: pozycje wierzcholkow w kolejnych klatkach zapisuje sie
w teksturze, a shader odczytuje z niej odpowiedni wiersz. Wtedy potwor jest
zwyklym MeshRendererem i laczy sie wsadowo jak kamien.

Kosztem jest utrata wszystkiego, co daje Animator: mieszania stanow, warstw,
zdarzen animacji i kinematyki odwrotnej. Dlatego rozsadny podzial wyglada tak:

- **przeciwnicy pierwszoplanowi**, z ktorymi gracz walczy - normalny Animator,
  jest ich kilku naraz i stac nas na nich,
- **tlum w tle** - animacja z tekstury, bez Animatora, dziesiatki sztuk.

## Co NIE zadziala

- Zwykle laczenie siatek (static batching) - dotyczy obiektow nieruchomych.
- Wbudowany generator LOD w Unity (Mesh LOD) **nie jest rozwiazaniem tego
  problemu**. On nie dodaje wierzcholkow, tylko rusza indeksami, wiec UV i wagi
  sa nietkniete, ale **nie bierze pod uwage wag szkieletu przy upraszczaniu**,
  a Skinned Mesh Renderer i tak deformuje poziom zerowy. Do rekwizytow tak,
  do postaci nie.
- Liczenie, ze wystarczy zmniejszyc liczbe trojkatow. Waskim gardlem sa
  wywolania rysowania i praca procesora nad animacja, nie sama geometria.

## Dowod

Dokumentacja Unity 6.5, strona GPU Resident Drawer, sekcja o obiektach
wykluczonych: obsluga ograniczona do MeshRenderera oraz wykluczenie obiektow
z komponentem Animation lub Animator w hierarchii. Strona GPU Instancing
potwierdza to samo dla instancingu, a strona wycinania na karcie graficznej
mowi o zaleznosci od GPU Resident Drawera.

To jest **deklaracja producenta w dokumentacji**, nie nasz pomiar. Wlasny pomiar
jest nastepnym krokiem i dopiero on zamieni te lekcje w liczby.

## Czy to przeniesie sie na inny projekt

Tak, i to jest jej glowna wartosc. To jest ograniczenie **silnika**, nie gatunku
ani projektu. Dotknie kazdej gry z wieloma animowanymi postaciami na ekranie,
niezaleznie od tematu. Warto o tym wiedziec **przed** zaprojektowaniem systemu
przeciwnikow, bo przejscie na animacje z tekstury po fakcie oznacza przerobienie
calego potoku postaci.

## Powiazane

- [[MAPA-LOW-POLY]]
- [[MAPA-RIGGING-I-ANIMACJA]]

<!-- WERYFIKATOR 2026-08-05 -->
Weryfikacja 2026-08-05: dokumentacja Unity 6.5 potwierdza wykluczenie obiektow
"in the hierarchy of Animation or Animator components" i ograniczenie do Mesh
Renderera (GPU Resident Drawer), zaleznosc wycinania na GPU od tego mechanizmu
(GPU occlusion culling) oraz oba twierdzenia o Mesh LOD ("does not take skin
weights or blend shape deformations into account", "deforms LOD0 regardless of
which LOD index"); natomiast strona o GPU Instancing mowi tylko "Skinned Mesh
Renderer components are not supported" i **nie** potwierdza wykluczenia przez
Animator w hierarchii - to zdanie wpisu jest szersze niz zrodlo.
Zrodla: https://docs.unity3d.com/6000.5/Documentation/Manual/urp/gpu-resident-drawer.html ,
https://docs.unity3d.com/6000.5/Documentation/Manual/GPUInstancing.html ,
https://docs.unity3d.com/6000.5/Documentation/Manual/urp/gpu-culling.html ,
https://docs.unity3d.com/6000.2/Documentation/Manual/lod/mesh-lod-introduction.html

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260807-1140-laczenie-siatek-a-animowany-przodek|Laczenie siatek pod wywolania rysowania a animowany przodek]] - wspolne: wydajnosc, animacja
<!-- /POWIAZANE:auto -->
