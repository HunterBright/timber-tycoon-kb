---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [skinning, linear-blend-skinning, rigging, blender, unity, humanoid, bind-pose, bramki]
date: 2026-08-07
status: draft
---

# Obrot dloni "wazony waga kosci" zawsze zapada nadgarstek - wzor to cos(A/2)

## Objaw

Dlonie postaci mialy byc obrocone o 90 stopni (wnetrze z "do przodu" na "do uda").
Trzy niezalezne proby daly ten sam efekt, opisany przez reżysera jako "dlonie sa
wykrecone": obrot kosci dloni w grze, obrot wierzcholkow siatki wazony waga kosci,
i ten sam obrot z odwroconym znakiem.

## Dlaczego to byla za kazdym razem TA SAMA proba

Skinning liniowy (LBS) to `v' = suma(w_i * M_i * v)`. Gdy jedna kosc stoi (macierz
jednostkowa), a druga obraca sie o kat A, wzor sprowadza sie do:

    v' = (1 - w) * v + w * (R * v)  =  lerp(v, R*v, w)

To jest cieciwa, nie luk. Punkt lezacy w promieniu r od osi po takiej operacji lezy
w promieniu `r * cos(A/2)` przy w = 0,5. Dla 90 stopni to 71%, dla dwoch takich
operacji zlozonych (siatka + komponent w grze) - okolo 45% pierwotnej grubosci.
Nadgarstek dosłownie zapada sie do polowy. Zaden dobor znaku ani kolejnosci tego nie
zmienia, bo wszystkie trzy proby liczyly ten sam lerp.

## Rozwiazanie: zepchnac obrot do POZY WIAZANIA i uzyc PRAWDZIWEGO obrotu

Poza wiazania nie powstaje ze skinningu, wiec wolno w niej zastosowac dowolne
przeksztalcenie. Obrot wokol osi jest izometria - zachowuje KAZDA odleglosc od tej
osi, wiec obwod nie zmienia sie ani o promil.

    p' = Rot(u, theta(s)) * (p - A) + A        # nigdy lerp

gdzie `u` to os konczyny, `s` polozenie wzdluz niej, a `theta(s)` narasta od zera
do pelnego kata. Trzy rzeczy, ktore decyduja o jakosci:

1. **Rozlozyc kat na CALY lancuch**, nie na ostatnie ogniwo. Scinanie powierzchni to
   `r * dtheta/ds`. Caly obrot 88 stopni na samym przedramieniu (20 cm dlugie, 12 cm
   grube) dal 40 stopni scinania rekawa; ten sam obrot rozlozony na ramie plus
   przedramie - 17,5 stopnia, czyli tyle, ile robi zywa reka przy pronacji.
2. **Przyrost kata odwrotnie proporcjonalny do promienia** (`dtheta/ds ~ 1/r`) daje
   scinanie STALE na calej dlugosci zamiast jednego pierscienia zagniecenia.
3. **Okno smoothstep na obu koncach** zeruje pochodna przy barku i przy nadgarstku.
   Bez niego powstaja dwa widoczne pierscienie.

Za ostatnim stawem czlon dostaje pelny kat SZTYWNO - tam deformacji ma nie byc wcale.

## Czego NIE robic

- **Nie obracac kosci** (roll). W pozie spoczynkowej nie przesuwa to ani jednego
  wierzcholka, a rozjezdza zapisany szkielet awatara i wymusza jego przebudowe.
- **Nie kompensowac w czasie gry.** Przy 4-kosciowym LBS `cos(A/2)` jest nieusuwalne.
  Przesuwanie skretu miedzy kosciami (`foreArmTwist`) przesuwa zgniecenie, nie zmniejsza go.
- **Nie zapominac o normalnych wlasnych.** Skrypt przesuwajacy wierzcholki, ktory nie
  rusza normalnych, zostawia powierzchnie cieniowana tak, jakby sie nie obrocila - na
  gladkim modelu czyta sie to jak zagniecenie, niezaleznie od geometrii.

## Bramki, ktore to pilnuja (kazda z udowodniona dzwignia)

| bramka | co mierzy | wynik poprawny | stara metoda (lerp) |
|---|---|---|---|
| sztywnosc bryly | dlugosci krawedzi miedzy wierzcholkami o TYM SAMYM kacie | 1,0000 - 1,0000 x | 0,29 - 3,57 x |
| obwod | sredni promien w plasterkach, przed vs po | 0,00% zmiany | 26,4% zapadniecia przy nadgarstku |
| scinanie | `atan(r * dtheta/ds)` na powierzchni | 17,5 st. | (nie dotyczy) |

**Pulapka przy stawianiu tej bramki:** pierwsza wersja bramki sztywnosci mierzyla
WSZYSTKIE krawedzie i wymagala, zeby zadna sie nie skrocila ("obrot moze tylko
wydluzyc"). To bylo zle postawione pytanie - skret ROZLOZONY ma pelne prawo skracac
krawedzie ukosne, bo ich konce dostaja rozne katy. Bramka oblewala wynik, przy ktorym
obwod nie zmienil sie ani o promil. Poprawna wersja mierzy tylko pary o identycznym
kacie, czyli faktyczne bryly sztywne. Tolerancja "prawie ten sam kat" tez nie dziala:
1 stopien roznicy na promieniu 6 cm to ponad milimetr, czyli 20% dlugosci krotkiej krawedzi.

## Meta-lekcja o miernikach

Miernik napisany w silniku (kat rolki dloni liczony z SVD chmury wierzcholkow) po
obrocie o 90 stopni **nie drgnal** (-82,9 -> -83,4 stopnia), mimo ze roznice widac na
renderze golym okiem. Dla niemal plaskiej dloni kierunek najmniejszej wariancji jest
niejednoznaczny i miernik aliasuje. Zanim uznasz pomiar za werdykt, sprawdz go na
parze PRZED/PO, o ktorej niezaleznie wiesz, ze sie rozni.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260531-1530-unity-humanoid-autorig-mirrored-foot|Crooked foot under Unity Humanoid = auto-rig copied the foot bind pose instead of mirroring it]] - wspolne: bind-pose, humanoid, blender
- [[20260725-2320-fartuch-skinning-srednia-dwoch-ud-daje-zero|Fartuch ważony po połowie na oba uda NIE RUSZA SIĘ przy chodzie]] - wspolne: skinning, rigging, blender
- [[20260717-0010-generated-rig-bone-axis-defect-skeleton-transplant|Rigi z generatorów AI (Hunyuan): osie kości rozjechane z frontem modelu = wykrzywiona stopa w retargecie; lek = przeszczep szkieletu w Blenderze]] - wspolne: humanoid, rigging, blender
- [[20260802-1620-humanoid-retarget-poza-wzorcowa|20260802-1620-humanoid-retarget-poza-wzorcowa]] - wspolne: humanoid, rigging
- [[20260805-2320-bone-heat-pada-na-siatkach-wielobrylowych|20260805-2320-bone-heat-pada-na-siatkach-wielobrylowych]] - wspolne: skinning, blender
- [[20260731-2200-slepa-dzwignia-debugger-bramek|20260731-2200-slepa-dzwignia-debugger-bramek]] - wspolne: bramki, blender
<!-- /POWIAZANE:auto -->
