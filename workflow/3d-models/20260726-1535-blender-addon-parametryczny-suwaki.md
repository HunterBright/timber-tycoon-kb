---
title: 'Parametryczny dodatek do Blendera: trzy pulapki, ktore kosztuja godzine kazda'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-26'
project: Kerf - Sawmill Tycoon
tags:
- blender
- addon
- python
- parametric
- ui
- bpy-props
- timers
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Parametryczny dodatek do Blendera: trzy pulapki, ktore kosztuja godzine kazda

## Kontekst

Budowa dodatku, w ktorym nie-programista rusza suwakami, a siatka przebudowuje sie
na zywo. Wzorzec przenosny na kazdy projekt, gdzie ktos ma "pobawic sie parametrami"
zamiast czekac na render od kogos innego.

Architektura, ktora sie sprawdzila: JEDEN plik z lista parametrow, z ktorego czytaja
i interfejs, i geometria. Interfejs generuje suwaki z tej listy, geometria dostaje
gotowe wymiary. Nie da sie ich rozjechac, bo nie ma dwoch zrodel.

## Pulapka 1: wspolny try/except kasuje CALY interfejs

Wlasciwosci (`bpy.props`) doklada sie do klasy `PropertyGroup` PRZED
`register_class`, czyli raz, przy starcie dodatku. Nie da sie ich dolozyc pozniej
bez ponownej rejestracji.

Skutek: jesli jeden `try` obejmuje import pliku z parametrami I plikow z geometria,
to blad skladni w geometrii kasuje wszystkie suwaki az do restartu Blendera.
Uzytkownik widzi pusty panel i nie ma jak nawet cofnac zmiany.

```python
# ZLE - jeden blad w geometrii zabiera 38 suwakow
try:
    import params, geometria_a, geometria_b
except Exception as e:
    BLAD = str(e)

# DOBRZE - parametry osobno, sa fundamentem interfejsu
try:
    import params
except Exception as e:
    BLAD_KRYTYCZNY = str(e)     # panel nie ma z czego powstac
    return
try:
    import geometria_a, geometria_b
except Exception as e:
    BLAD_MIEKKI = str(e)        # suwaki zostaja, brakuje tylko wyniku
```

Test, ktory to wylapal: rejestracja dodatku, gdy plikow geometrii jeszcze NIE MA
na dysku. Warto go miec, bo to takze prawdziwy stan podczas rozwoju narzedzia.

## Pulapka 2: przebudowa siatki wprost w `update` suwaka

`FloatProperty(update=...)` odpala sie w trakcie obslugi interfejsu. Kasowanie
i tworzenie obiektow w tym momencie to grzebanie w danych, gdy Blender rysuje okno.
Do tego przeciagniecie suwaka wysyla kilkadziesiat zdarzen na sekunde i kazde
buduje model od zera.

Rozwiazanie zalatwia oba problemy naraz - odlozenie przez zegar z flaga:

```python
_czeka = False

def _odlozona_przebudowa():
    global _czeka
    _czeka = False
    przebuduj(bpy.context)
    return None            # None = nie powtarzaj

def _na_zmiane(self, context):
    global _czeka
    if _czeka:
        return             # sklej wszystkie zdarzenia w jedno
    _czeka = True
    bpy.app.timers.register(_odlozona_przebudowa, first_interval=0.05)
```

Uzywac WYLACZNIE API danych (`bpy.data.meshes.new`, `bpy.data.objects.remove`),
nigdy `bpy.ops.*` - operatory zaleza od kontekstu, ktorego w zegarze nie ma.

## Pulapka 3: nazwy w enumach zmieniaja sie miedzy wersjami

Nazwa silnika renderowania przeszla droge `BLENDER_EEVEE` -> `BLENDER_EEVEE_NEXT`
-> z powrotem `BLENDER_EEVEE` (5.x). Przypisanie nieistniejacej nazwy rzuca
`TypeError` i zabija skrypt.

Nie zgadywac po numerze wersji. Pytac Blendera, co ma:

```python
dostepne = {i.identifier for i in
            bpy.types.RenderSettings.bl_rna.properties["engine"].enum_items}
for kandydat in ("BLENDER_EEVEE_NEXT", "BLENDER_EEVEE", "BLENDER_WORKBENCH"):
    if kandydat in dostepne:
        sc.render.engine = kandydat
        break
```

Ta sama zasada dotyczy kazdego enuma w bpy oraz gniazd wezlow (`Specular` kontra
`Specular IOR Level` w Principled BSDF).

## Osobno: instalacja bez klikania

`bpy.ops.preferences.addon_install` + `addon_enable` + `wm.save_userpref`
uruchomione w `blender -b` (BEZ `--factory-startup`, bo to ma trafic do prawdziwych
ustawien) instaluja dodatek za uzytkownika.

Haczyk: instalowana jest KOPIA pliku w katalogu dodatkow. Zeby poprawki geometrii
dzialaly bez ponownej instalacji, dodatek powinien dopisywac katalog projektu do
`sys.path` i importowac geometrie stamtad, plus miec guzik "Przeladuj kod"
wolajacy `importlib.reload`. Wtedy reinstalacji wymaga tylko zmiana samego panelu.

## Wniosek przenosny

Gdy narzedzie ma trafic do osoby nietechnicznej, interfejs musi przezyc awarie
tego, co obsluguje. Panel, ktory znika przy bledzie, jest gorszy od panelu, ktory
mowi "geometria sie nie wczytala" i zostawia suwaki na miejscu.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[blender-headless-python-generation|Blender Headless Python Script Generation]] - wspolne: python, blender
- [[20260704-1732-blender-linked-basecolor-recolor|Recoloring a Blender material whose Base Color is LINKED does nothing via default_value]] - wspolne: python, blender
- [[20260725-1930-blender-pixels-buffer-not-converted-to-srgb-on-png-save|Blender nie przelicza `image.pixels[]` na sRGB przy zapisie PNG]] - wspolne: python, blender
- [[20260730-1710-blender-materials-clear-resets-face-indices|Mesh.materials.clear() zeruje material_index na ściankach]] - wspolne: python, blender
- [[20260612-1845-blender-9slice-ui-sprites|Blender-rendered 9-slice-ready UI sprites (3D panel → ortho render → Unity Sliced sprite)]] - wspolne: ui, blender
<!-- /POWIAZANE:auto -->
