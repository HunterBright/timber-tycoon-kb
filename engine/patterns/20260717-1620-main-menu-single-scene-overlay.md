---
title: Menu glowne jako nakladka w jednej scenie (bez sceny menu)
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- main-menu
- single-scene
- timescale
- test-automation
- bootstrap
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Menu glowne jako nakladka w jednej scenie (bez sceny menu)

## Problem

Gra ma jedna scene, managery zyja w niej bez DontDestroyOnLoad, a potrzebne jest
menu glowne przy starcie. Osobna scena menu wymagalaby duplikacji bootstrapu
wszystkich managerow albo refaktoru na DontDestroyOnLoad. Do tego projekt ma
zautomatyzowane testy buildu (sonda CLI), ktore musza startowac prosto do gry.

## Wzorzec

1. **Menu = ekran UI w scenie gry**, pokazywany nad ujeciem z osobnej, tworzonej
   w runtime kamery. Gra laduje sie CALA przed menu, wiec "Nowa gra"/"Kontynuuj"
   startuje natychmiast - zero ekranu ladowania, zero drugiej sceny, zero edycji
   (binarnej) sceny.
2. **Przejecie startu**: hook `[RuntimeInitializeOnLoadMethod(AfterSceneLoad)]`
   biegnie po Awake wszystkich obiektow, PRZED Start i przed pierwsza klatka -
   przelaczenie stanu gry na MainMenu (timeScale=0, pauza audio, mapa inputu UI)
   jest niewidoczne i nieslyszalne dla gracza. Czarna kurtyna (CanvasGroup,
   unscaled time) przykrywa pierwsze klatki.
3. **Stan serializowany w scenie zostaje "Playing"** - menu wlacza sie OPT-IN
   w runtime. Dzieki temu sciezka testowa jest IDENTYCZNA jak przed istnieniem
   menu (zero ryzyka regresji w automatach).
4. **Centralna bramka trybow testowych** (statyczna klasa, np. TestRunGate):
   jedna lista flag CLI + plikow-flag; menu pyta ja w hooku i pomija sie samo.
   Opcje GRACZA (np. -notelemetry) celowo POZA lista.
5. **Kamera menu**: nowy GameObject w runtime, kopiuje parametry kamery gracza,
   dryfuje miedzy pozami A/B po unscaled time (swiat zamrozony, ruch kamery daje
   zycie). Kamere gracza wylaczac PER KOMPONENT Camera (AudioListener zostaje -
   dokladnie jeden). Rig dostaje tag MainCamera, bo skrypty w Update siegaja po
   Camera.main i dostalyby null.
6. **Sonda buildu dostaje check**: "menu pominelo tryb testowy" (stan Playing,
   timeScale 1) jako PIERWSZA sekcja bez skalowanych czekan + czerwona probka
   (`-menuredproof` otwiera menu celowo -> check musi FAIL) na TYM SAMYM buildzie.

## Pulapki

- `VerticalLayoutGroup` z `childControlHeight=false` NIE respektuje
  `LayoutElement.preferredHeight` - wysokosc elementu (logo!) trzeba ustawic
  wprost na RectTransform.sizeDelta.
- Okna "wspoldzielone" (Ustawienia/Tworcy otwierane i z pauzy, i z menu) potrzebuja
  "adresu powrotu" (Open(Action onClose)), inaczej Close() wraca twardo do pauzy.
- Skroty klawiszowe gameplayu (ekwipunek, kolo narzedzi) czesto NIE sa bramkowane
  stanem gry i otworza sie NAD menu - audyt wszystkich `GetKeyDown` w Update.
- HUD to zwykle KILKA niezaleznych widgetow (pieniadze, questy, reputacja, kompas),
  czesc tworzona PO menu w kolejnosci canvasu (rysuja sie NAD menu) i czesc
  z wlasnym Start(), ktory wygrywa wyscig z chowaniem - kazdy potrzebuje SetVisible
  z guardem "nie pokazuj, gdy menu otwarte".
- Zmiana productName przenosi persistentDataPath - zapisy graczy "znikaja".
  Migracja: `[RuntimeInitializeOnLoadMethod(BeforeSceneLoad)]`, KOPIUJ (nie przenos),
  idempotentnie.

## Kiedy stosowac

Gry single-scene z managerami w scenie. Przy wielu scenach / DontDestroyOnLoad
klasyczna scena menu moze byc czystsza.

## Uzupelnienie (iteracja 2): zywy swiat i scenografia

- Zywe tlo menu NIE wymaga przerabiania shaderow: jesli woda/trawa/efekty siedza na
  `_Time` (skalowanym), wystarczy w stanie menu ustawic timeScale=1 zamiast 0 i dolozyc
  bramki IsPlaying TYLKO tam, gdzie realnie postepuje progresja (audyt: wiekszosc
  systemow juz jest bramkowana; typowe dziury to cykl dnia i wzrost roslin).
- Pulapka subskrypcji: system pauzujacy cykl dnia subskrybowal event stanu w Start(),
  a menu przelacza stan w AfterSceneLoad (przed Start) - po subskrypcji trzeba
  APLIKOWAC stan biezacy, nie czekac na nastepny event.
- Ambient w menu: AudioListener trzeba PRZENIESC na kamere menu (uszy zostaja na
  spawnie gracza - dzwiek 3D "przy ujeciu" bylby niemy).
- Scenografia (menu pokazuje swiezy swiat, a chcemy "rozwinieta" baze): wizualne klony
  ukrytych obiektow - Instantiate nieaktywnego zrodla, strip WSZYSTKICH skryptow/
  colliderow/fizyki PRZED SetActive (zero Awake/rejestracji), niszczone przy starcie gry.
  Obiekty chowane na czas menu przywracac PRZED wczytaniem zapisu (zapis = ostatnie slowo).

## Uzupelnienie (iteracja 6): scenografia = "jak w grze", nie "jak ladnie"

Gdy menu pokazuje wnetrze bazy/warsztatu jako tlo, pokusa jest skomponowac
"idealna ekspozycje" - poustawiac regaly/przedmioty na srodku, zeby ladnie
wpadaly w kadr. TO BLAD. Rezyser/gracz oczekuje, ze tlo menu odwzorowuje
PRAWDZIWY stan bazy po odblokowaniu (kazdy obiekt na swojej docelowej pozycji
z gry). Regula: dekoracja menu ma byc "jak w grze po rozbudowie", nie
"przearanzowana wystawa sklepowa". Technicznie: odslaniaj/wypelniaj obiekty
NA ICH ORYGINALNYCH POZYCJACH (reveal/fill), a kamere dostraja do tego, co jest -
nie przesuwaj swiata pod kamere. Jedyny wyjatek to obiekty, ktore w swiezej
scenie w ogole nie istnieja na docelowej pozycji (np. meble, ktore gracz
dopiero wytwarza) - te stawiamy na ICH przyszlych miejscach (piedestaly wystawy),
zmierzonych z gry, nie "gdzie pasuje do kadru".

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260722-1850-single-scene-return-to-menu-survivors|Powrot do menu glownego w grze jednoscenowej: co przezywa przeladowanie sceny]] - wspolne: main-menu, single-scene
<!-- /POWIAZANE:auto -->
