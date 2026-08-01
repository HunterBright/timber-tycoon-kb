---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: workflow/lessons
tags: [qa, bramki, sonda, dzwignia, testowanie, blender, proceduralne]
date: 2026-07-31
status: draft
severity: high
---

# Ślepa dźwignia to nie porażka dowodu - to debugger bramki

## Kontekst
Generator NPC (Blender, bmesh): bramka mierząca ciągłość krawędzi ubrania
dostała dźwignię-dowód (celowe złamanie geometrii przed pomiarem; budowa MUSI
oblać). Dźwignia trzy razy z rzędu "nie zadziałała" - budowa przechodziła na
zielono z wszczepioną 15 mm wadą i ZAPISYWAŁA plik.

## Lekcja
Każde z trzech ślepych przejść obnażyło INNĄ, realną dziurę bramki, której
żaden przegląd kodu nie wykrył:
1. **Dziura w danych**: okno wyboru celu trafiło w przęsło siatki bez
   wierzchołków (7 cm luki w linii krawędzi) - dźwignia nie znalazła celu
   i "cicho" przeszła.
2. **Ślepa strefa miary**: pomiar liczył tylko węzły o dokładnie 2 sąsiadach;
   dźwignia trafiła węzeł rozgałęzienia (4 sąsiadów) - wada mierzalna, ale
   pomijana filtrem.
3. **Stare dane pomocnicze**: sonda klasyfikowała ścianki buforem policzonym
   PRZED pętlą deformacji - mierzyła finalną siatkę starą mapą, przez co
   widziała innych sąsiadów niż istnieją.

Wniosek: dowód dźwigni nie jest formalnością do odhaczenia. Ślepa dźwignia =
znaleziona dziura bramki. Procedura przy ślepym przejściu: (a) NIE poprawiać
dźwigni "na czuja", (b) najpierw replika pomiaru offline na pliku z wadą
(wada w środku = idealny materiał badawczy), (c) porównać replikę z produkcyjnym
pomiarem - różnica wskazuje dziurę, (d) naprawić BRAMKĘ, nie tylko dźwignię.

## Pułapka dodatkowa
Ślepy dowód z zapisem pliku zostawia na dysku artefakt Z WSZCZEPIONĄ WADĄ -
po każdym nieudanym dowodzie sprawdź, czy budowa nie zapisała wyniku, i po
naprawie przebuduj artefakt na czysto.

## Kiedy stosować
Każdy nowy check w bramkach buildowych / sondach proceduralnych, w dowolnym
silniku: check bez dowiedzionego trybu porażki niczego nie pilnuje, a dowód,
który przeszedł na zielono, jest cenniejszy diagnostycznie niż ten, który
od razu obleje.

## Uzupełnienie z werdyktu niezależnego sędziego (ta sama sesja)
1. **Naprawiacz nie może optymalizować miary sędziego.** Gdy mechanizm
   naprawczy (prostowanie krawędzi) celowo dzieli z bramką tę samą definicję
   pasm i tę samą heurystykę, bramka przestaje mierzyć jakość - mierzy tylko,
   czy jej własny naprawiacz zbiegł. Wniosek: obok miary optymalizowanej
   trzymaj drugą miarę o INNEJ definicji (albo oceniaj artefakt wizualnie),
   inaczej wartość "pod progiem" nic nie dowodzi.
2. **Dowód dźwigni musi jechać na kodzie, który idzie w świat** - sekwencja:
   baza zielona -> dźwignia -> czerwono, na tym samym commicie. Dowód zrobiony
   na starszej wersji kodu, albo na budowie i tak już czerwonej z innego
   powodu, nie dowodzi bramki w wydanym kodzie.
3. **Uczciwość streszczenia dowodów**: sędzia porównuje streszczenie z plikami
   na dysku; pominięty przebieg (nawet bez złej woli) podważa zaufanie do
   całej reszty opisu.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0500-gestszy-pomiar-odslania-dlug|20260801-0500-gestszy-pomiar-odslania-dlug]] - wspolne: proceduralne, bramki, blender
- [[build-is-the-only-truth-editor-lies|Edytora nie da sie oszukac, zeby udawal build]] - wspolne: sonda, qa
- [[gate-must-have-provable-failure-mode|Bramka bez udowodnionego trybu porazki niczego nie pilnuje]] - wspolne: sonda, qa
- [[20260726-1810-ciagla-powloka-zamiast-osobnych-bryl|"Zle przyklejone konczyny" to nie blad ustawienia, tylko blad architektury]] - wspolne: proceduralne, blender
- [[20260727-1309-naprawiony-suwak-uniewaznia-strojenie|Naprawa suwaka, ktory po cichu klamal, uniewaznia CALE wczesniejsze strojenie]] - wspolne: proceduralne, blender
- [[20260731-2115-bramka-ktora-istnieje-ale-nie-odpala-sie-dla-wiekszosci-obiektow|Bramka, ktora istnieje, ale nie odpala sie dla wiekszosci obiektow]] - wspolne: bramki, blender
<!-- /POWIAZANE:auto -->
