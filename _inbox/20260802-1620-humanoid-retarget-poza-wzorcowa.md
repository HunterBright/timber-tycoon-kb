---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [unity, humanoid, retargeting, mixamo, rigging, animacja, npc]
date: 2026-08-02
status: draft
severity: wysoka
---

# Poza wzorcowa rzadzi cala animacja humanoidalna - i zwykle to ona jest zepsuta

## Objaw

NPC chodzi zle: lewa stopa wykrzywiona (prawa zawsze dobra), dlonie wnetrzem do przodu,
rece odstajace od ciala. Objaw powtarza sie na **kazdym** modelu wgranym do gry, takze na
postaci, ktora dziala od miesiecy.

## Dlaczego szukanie w modelu to strata czasu

Unity przy animacji humanoidalnej przeklada ruch przez pozy wzorcowe:

    poza wzorcowa ZRODLA -> przestrzen miesni -> poza wzorcowa CELU

Klip nie zawiera pozycji kosci, tylko **roznice wzgledem pozy wzorcowej postaci, na ktorej
go zrobiono**. Kazdy blad w tej pozie dokłada sie do KAZDEJ klatki u KAZDEJ postaci
grajacej ten klip. U zrodla blad sie znosi (zrodlo i cel to ten sam szkielet), u nas nie
ma sie z czym znosic.

## Co bylo faktycznie zepsute (klip X Bot z Mixamo)

Zmierzone w pozie wzorcowej zapisanej w awatarze:

| element | wartosc | powinno byc |
|---|---|---|
| lewa kostka wobec prawej | rozjazd 25 st. (kolano 13 st.) | 0 - cialo jest symetryczne |
| kierunek lewej stopy vs lustro prawej | **38,9 st.** | ~0 |
| obrot wnetrza dloni | **39,7 / 40,6 st.** | 0 (w pozie T dlon patrzy w dol) |

**Model z Mixamo NIE jest wzorcem pozy T.** Przy kazdym nowym klipie sprawdz oba miejsca,
zanim go uzyjesz.

## Naprawa

Zapisac poprawione obroty w `ModelImporter.humanDescription.skeleton`:
- **stopy**: policzyc poze prawej nogi w przestrzeni swiata, odbic wzgledem plaszczyzny
  symetrii, zapisac jako lewa (odbicie macierzy `S*M*S`, wyznacznik zostaje +1),
- **dlonie**: obrocic kosc dloni wokol osi PRZEDRAMIENIA, az wnetrze patrzy w dol.

Wynik (pomiar w trybie gry, 1600 klatek, skret lewej stopy):
przed +11,4 st. -> po -2,5 st., przy wzorcu -2,8 st. Naprawia wszystkie postacie naraz.

## Co NIE dziala (sprawdzone)

- **Odpinanie palcow stop od awatara** - postac z odpietymi i postac ze zmapowanymi
  palcami mialy identyczna wade. Zabieg tylko odbiera animacje palcow.
- **"Enforce T-Pose"** (`AvatarSetupTool.MakePoseValid`) - niesymetria 10,4 -> 10,3 st.,
  czyli nic. Ta funkcja sprowadza poze do dozwolonego zakresu, nie wyrownuje stron.

## Wybor MIEJSCA poprawki

- poprawka w pozie wzorcowej **KLIPU** -> dotyczy wszystkich postaci grajacych ten klip,
- poprawka w pozie wzorcowej **POSTACI** (`humanDescription.skeleton` jej FBX) -> dotyczy
  tylko jej, reszta gry nietknieta.

Wybieraj wedlug tego, kogo wada dotyczy. Wada u jednej postaci naprawiana w klipie psuje
te postacie, ktore byly dobre.

## Osobna pulapka: kosc ramienia poza tulowiem

Przy budowaniu rigu kosc ramienia wyladowala 10 cm ZA krawedzia tulowia (0,297 m przy
krawedzi 0,198 m; u dzialajacej postaci szpara wynosi 0,023 m).

**W pozie spoczynkowej tego nie widac** - siatka stoi na miejscu, wiec zaden render
statyczny ani audyt bindpose tego nie zlapie. Ale animacja obraca konczyne WOKOL kosci,
wiec reka krazyla wokol punktu wiszacego w powietrzu obok ciala i odstawala jak przy
niesieniu czegos pod pacha.

**Bramka:** staw barkowy jako procent wzrostu. Wzorzec 12,3%, wadliwy 16,6%.
Analogicznie dla biodra i kolana.

## Reguly do zapamietania

1. Gdy objaw wystepuje na KAZDYM modelu, przyczyna jest w czesci WSPOLNEJ (klip, szkielet,
   awatar), nie w modelu. Pierwszy ruch: sprawdz, czy postac, ktora dziala, ma to samo.
2. Postac zrodlowa klipu wstaw do sceny testowej jako wzorzec - gra wlasny klip na wlasnym
   szkielecie, wiec pokazuje, jak ten ruch wyglada bez przekladania.
3. Przekladanie animacji humanoidalnej dziala WYLACZNIE w trybie gry. Render z edytora
   pokazuje poze spoczynkowa i klamie.
4. Wady rotacji nie widac w pozie statycznej. Mierz w RUCHU i porownuj lewa strone z prawa -
   chod czlowieka jest symetryczny, wiec kazda trwala roznica to wada.
