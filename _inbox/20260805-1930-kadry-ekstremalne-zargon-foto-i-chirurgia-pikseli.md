---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [ai-image, qwen-image-edit, referencje-3d, hunyuan, multi-view, prompt]
date: 2026-08-05
status: draft
---

# Kadry ekstremalne wymusza zargon fotograficzny, a poprawki rozmiaru - chirurgia pikseli

## Problem
Robienie kompletu widokow postaci (przod/boki/tyl/gora/dol) pod generator bryly 3D
(Hunyuan "Multiple Images") jednym modelem obrazkowym (Qwen-Image-Edit 2509).

## Lekcja 1: kadru z gory/z dolu nie wymusisz opisem potocznym
14 prob "show from above / obroc kamere w dol / chinska komenda LoRA" - wszystko wracalo
widokiem z przodu (zmierzony skrot perspektywiczny 1-6% zamiast >30%).
Zadzialal dopiero ZARGON FOTOGRAFICZNY: "Nadir shot, top-down 90 degrees, aerial drone
photo" -> skrot 74% od pierwszego ziarna. Modele obrazkowe znaja te frazy z opisow zdjec.
Widok od spodu przez ODWROTKE: wejscie do gory nogami + ten sam przepis nadir + wynik
odwrocony z powrotem (dla postaci symetrycznej lewo-prawo odbicie nic nie psuje).

## Lekcja 2: sprawdz, czy sam nie sabotujesz polecenia
Tekst pilnujacy zgodnosci widokow ("cala postac od glowy do stop w kadrze, stopy plasko
na ziemi") wprost ZABRANIA ujecia z gory. Model wybieral posluszenstwo wobec tego tekstu.
Przy ujeciach ekstremalnych trzymajacy tekst musi miec wersje bez wymagan kadru i stop.

## Lekcja 3: edycja "zmien tylko X, rozmiar zostaw" to loteria - tnij w pikselach
Prosba o przebudowe dloni dala ladne palce, ale dlon +37% (odrzucona przez rezysera).
Prosba "tylko poglebic rowki, nic wiecej" - model albo nie robil nic, albo demolowal
obrazek (0 zdanych na 6). Zadanie bylo geometryczne, wiec wygral zwykly kod:
wykryj ciemne linie rowkow, namaluj wzdluz nich tlo (scipy + PIL). Rozmiar +0%,
idealna powtarzalnosc na obu rekach. Generatywna edycja do ZMIANY tresci,
deterministyczna do ZACHOWANIA tresci.

## Lekcja 4: male detale edytuj w wycietym kadrze
Dlon zajmujaca kilka procent kadru 1024 jest dla modelu "detalem" - poprawki nie wchodza.
Wyciecie samego kadru dloni do pelnych 1024 px sprawia, ze ta sama poprawka dziala.
(Ale uwaga na lekcje 3 - wklejka wraca w innym rozmiarze, jesli nie pilnuje jej bramka.)

## Bramki, ktore to wszystko trzymaly
- skrot perspektywiczny = 1 - (wysokosc sylwetki / wysokosc w przodzie); gora OK przy >=30%
- liczba palcow = odcinki sylwetki na linii skanu (progi: palec >=6 px, szczelina >=4 px)
- rozmiar dloni = wysokosc bryly w pasie skrajnych 17% szerokosci ciala, tolerancja 10%
- winieta tla: tlo liczone PER WIERSZ z 12 skrajnych kolumn, nie jeden kolor
- kierunku profilu (nos w lewo/prawo) NIE mierzylismy wiarygodnie - centroid skory klamie
  (lapie dlon przy twarzy); kierunek sprawdzac okiem
