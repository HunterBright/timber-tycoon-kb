---
title: Laczenie siatek pod wywolania rysowania a animowany przodek
type: pattern
status: draft
confidence: medium
verified:
tags: [blender, unity, wydajnosc, animacja, draw-calls, low-poly, headless]
date: 2026-08-07
project: Kerf - Sawmill Tycoon
source: https://github.com/czlonkowski/blender-web-3d-skill (plugins/blender-web-3d/skills/blender-web-3d/references/blender-scripting.md, MIT)
applies_to: [blender-5.x, unity-6.x, pipeline]
suggested-category: pipeline/patterns
---

# Laczenie siatek pod wywolania rysowania a animowany przodek

## Kiedy stosowac

Model jest gotowy pod wzgledem ksztaltu i **zaczyna kosztowac za duzo wywolan
rysowania** (draw calls). Klasyczny przypadek: asset zlozony z kilkuset drobnych
obiektow, powielony kilka razy w scenie, daje tysiace wezlow siatki. Wtedy
scala sie rodzenstwo w jedna siatke na grupe.

To jest krok **po** ustabilizowaniu ksztaltu, nigdy w trakcie modelowania.

## Kroki

1. Pogrupuj rodzenstwo po trojce: **ten sam material, ta sama sygnatura warstw
   UV, ten sam korzen animacji**.
2. Scal kazda grupe w jedna siatke, przenoszac wierzcholki do przestrzeni
   lokalnej docelowego rodzica **przez zlozony lancuch `matrix_local`**, a nie
   przez `matrix_world`.
3. Obiekty zablokowane wylacznie przez modyfikatory mozna scalic po wypieczeniu
   siatki wynikowej: `bpy.data.meshes.new_from_object(obj.evaluated_get(depsgraph), ...)`.
4. Porownaj render przed i po. Zmiana ma byc **zerowa wizualnie**.

## Czego NIGDY nie scalac

To jest sedno tego wpisu.

- obiektow z wlasna akcja, sciezkami NLA, ksztaltami mieszanymi albo sterownikami
- **obiektow, ktore maja ANIMOWANEGO PRZODKA** - trzeba przejsc cala sciezke
  rodzicow w gore; jesli ktorykolwiek przodek niesie akcje, siatka musi zostac
  pod nim
- siatek z wlasnymi rozdzielonymi normalnymi (scalanie je niszczy)
- obiektow wielomaterialowych, pustych obiektow powielajacych i wezlow
  kotwiczacych hierarchie, ktore maja dzieci

## Dlaczego to dziala

Wywolanie rysowania kosztuje za kazdy wezel siatki, niezaleznie od liczby
trojkatow. Scalenie rodzenstwa o wspolnym materiale zamienia setki tanich
wezlow na jeden, bez zmiany tego, co widzi gracz.

Regula o animowanym przodku wynika z tego, gdzie siedzi transformacja: po
scaleniu siatka **przestaje byc dzieckiem animowanego obiektu** i zostaje na
zawsze w pozie spoczynkowej, podczas gdy jej rodzic dalej sie rusza. Blad nie
zglasza sie zadnym komunikatem - widac go dopiero na renderze albo w grze.

## Koszty i kompromisy

- Model po scaleniu jest **trudniejszy do dalszej edycji**, wiec trzyma sie
  wersje sprzed scalenia.
- Sciezka `matrix_world` bywa **nieaktualna** dla obiektow zyjacych wylacznie
  wewnatrz powielonych kolekcji - stad wymog liczenia przez `matrix_local`.
- Zysk jest w wezlach, nie w trojkatach. Jesli problemem sa trojkaty, to jest
  zadanie dla poziomow szczegolowosci, a nie dla scalania.

## Warianty

Przy tlumie postaci w Unity ten wzorzec **nie wystarczy**, bo GPU Resident
Drawer i tak wyklucza obiekt, gdy gdziekolwiek w hierarchii siedzi Animator.
Tam trzeba przeniesc animacje do tekstury. Scalanie dziala na tym, co nieruchome:
otoczenie, budynki, rekwizyty, statyczne czesci potwora.

## Dowod, ze zadzialalo u nas

**Jeszcze nie zadzialalo u nas - to jest wpis do sprawdzenia.**

Dowod cudzy, ale konkretny: autor wzorca podaje redukcje z 4825 do okolo 1180
wezlow siatki przy zerowej zmianie wizualnej, na projekcie, ktory wydal.
Regule o animowanym przodku opisuje jako **skutek prawdziwego bledu**:
obrecze anteny zostaly wypieczone jako nieruchome, podczas gdy ich zawieszenie
sie obracalo.

Do sprawdzenia u nas na pierwszym modelu skladajacym sie z wielu czesci.

## Powiazane

- [[MAPA-PIPELINE-BLENDER-UNITY]]
- [[MAPA-LOW-POLY]]
- [[MAPA-WALKA-EFEKTY-PVE-PVP]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-1120-instancing-unity-omija-animowane-postacie|Szybkie rysowanie w Unity 6 omija wszystko, co ma Animator]] - wspolne: wydajnosc, animacja
- [[20260807-1155-skrypt-zarzadzany-blender-bezokienkowy|Skrypt zarzadzany - dyscyplina edycji Blendera bezokienkowo]] - wspolne: headless, blender
- [[blender-headless-python-generation|Blender Headless Python Script Generation]] - wspolne: headless, blender
- [[20260610-1820-blender-mcp-failure-headless-fallback|blender-mcp bridge failure modes + headless CLI fallback]] - wspolne: headless, blender
- [[20260731-1050-rowne-krawedzie-ubran-bisect-plane|20260731-1050-rowne-krawedzie-ubran-bisect-plane]] - wspolne: low-poly, blender
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: low-poly, blender
<!-- /POWIAZANE:auto -->
