---
title: Zielona tablica bramek nie dowodzi, ze cos wyglada dobrze
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-26'
project: Kerf - Sawmill Tycoon
tags:
- testy
- bramki
- geometria
- ocena-wizualna
- agenci
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Zielona tablica bramek nie dowodzi, ze cos wyglada dobrze

## Co probowalismy

Proceduralny generator postaci, rozwijany rundami: opis wady -> poprawka -> tablica
kontroli liczbowych. Kontrole byly dobre i uczciwe (kazda z udowodnionym trybem
porazki): szczelnosc, symetria co do bitu, jedna spojna bryla, budzet trojkatow,
wysuniecie brody, szerokosc barkow, zasieg kazdego suwaka.

## Dlaczego nie zadzialalo

**Trzy rundy z rzedu skonczyly sie tablica w calosci zielona i renderem gorszym
niz poprzednio.** Za kazdym razem inna wada: raz ostry grzbiet przez srodek twarzy,
raz gleboke bruzdy po bokach, raz pozioma polka pod zuchwa. Zadna kontrola tego nie
widziala, bo zadna nie mierzyla tego, co decyduje o wygladzie.

Podobnie przy gestosci siatki: podniesienie liczby segmentow o 20% przeszlo
155 kombinacji suwakow bez jednego bledu, a postac wygladala jak worek. Ksztalty byly
dostrojone do poprzedniej gestosci i nikt tego nigdzie nie zapisal.

## Wniosek

Kontrola liczbowa pilnuje **poprawnosci** (czy da sie tego uzyc), nie **jakosci**
(czy to dobrze wyglada). To sa dwa rozlaczne zbiory. Rozwijanie czegos wizualnego
wylacznie przez bramki daje obiekt bezbledny i brzydki.

## Co robic zamiast

1. **Render jest bramka, nie zalacznikiem.** Runda nie jest skonczona, dopoki ktos
   nie spojrzal na obrazek. Jesli wykonawca (czlowiek albo agent) pracuje na slepo,
   trzeba go zmusic do wyrenderowania i opisania WLASNYMI slowami, co widzi -
   inaczej raportuje tabelke i idzie dalej.
2. **Gdy render pokazuje wade, ktorej bramka nie zlapala, dopisz bramke.** Przyklad:
   pilnowalismy roznicy miedzy sasiednimi PIERSCIENIAMI, ale nic nie pilnowalo roznicy
   miedzy sasiednimi PUNKTAMI w jednym pierscieniu - i to wlasnie ona robila bruzdy.
3. **Kopia zapasowa PRZED kazda runda.** Bez tego "poprzednia wersja byla lepsza"
   jest zdaniem bez pokrycia. Stracilismy tak jedna lepsza wersje glowy.
4. **Zwezaj zakres rundy, gdy poprawki zaczynaja psuc.** Runda "napraw te dwie rzeczy
   i NIE dotykaj niczego innego" udala sie tam, gdzie rundy "popraw anatomie"
   psuly cos nowego.

## Diagnoza warta zapamietania

Powtarzajacy sie mechanizm tych wad: **cecha o zbyt waskim zasiegu**. Przy 20 punktach
na obwodzie przekroju kosc policzkowa rozlozona na jeden punkt daje blizne, a na piec
punktow - kosc policzkowa. Ta sama objetosc, inny odbior. Gdy proceduralny detal czyta
sie jak uszkodzenie, najpierw sprawdz, ile wierzcholkow go niesie, zanim zaczniesz
zmieniac jego glebokosc.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260727-1422-bramka-musi-umiec-zaliczyc-nie-tylko-oblac|Bramka musi mieć udowodniony tryb ZALICZENIA, nie tylko PORAŻKI]] - wspolne: testy, bramki
- [[20260728-1500-bramka-ponad-sufitem|Prog bramki ponad zmierzonym sufitem zamienia kazda runde w porazke]] - wspolne: testy, bramki
- [[20260731-0930-bramka-mierzaca-srodek-bryly-oblewa-ksztalty-niesymetryczne|Sprawdzian celujący w środek bryły oblewa kształty, których masa jest przesunięta]] - wspolne: testy, bramki
<!-- /POWIAZANE:auto -->
