---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [solver, iteracja, korekta, zbieznosc, geometria]
date: 2026-07-31
severity: high
status: draft
---

# Krok naprawczy PO zbiegnięciu iteracyjnego solvera psuje jego rozwiązanie

## Objaw
Iteracyjna korekta odstępów warstw (pchnięcia wierzchołków do marginesu,
pętla do zera poprawek) zbiega czysto, po czym bramka (sonda) oblewa z
przebiciem -25.7 mm. Winowajca: JEDNORAZOWE przycięcie rąbka wykonane PO
pętli - przesunęło wierzchołki, których pozycje solver już uzgodnił.

## Zasada
Każdy krok mutujący geometrię po zbiegnięciu solvera unieważnia jego
gwarancje. Nie wolno "dosztukować" poprawki po pętli - krok trzeba WŁĄCZYĆ
DO PĘTLI: jego ruchy liczą się do sumy poprawek, a pętla kończy się dopiero,
gdy solver ORAZ krok są w spoczynku. Jeśli kierunki ruchów solvera i kroku
są ~ortogonalne (pchnięcia poziome vs przycięcie pionowe), zbieżność kosztuje
1-2 dodatkowe przejścia.

## Wariant drugi: wygładzanie pola pchnięć
Surowe pchnięcia per wierzchołek (sąsiedzi stoją) robią z tkaniny poszarpane
łachmany - kilkumilimetrowe radialne schodki. Rozwiązanie w tej samej pętli:
na końcu każdego przejścia rozmyć część pchnięcia (30-40%) na sąsiadów,
jednolicie w bazie i wszystkich kluczach. Warunek bezpieczeństwa: margines
pchnięcia x (udział zachowany) MUSI zostać powyżej progu bramki
(np. 4.5 mm x 0.6 = 2.7 mm > 1.5 mm), inaczej wygładzanie i dopychanie
oscylują. Strefa martwa (poprawiaj tylko realne wnikania) zapobiega młócce.

## Jak rozpoznać ten błąd
Log pokazuje zbiegniętą pętlę (0 poprawek), a bramka mierzy dokładnie tę
strefę, którą ruszał post-krok. Sprawdź, co jeszcze dotyka geometrii między
ostatnim przejściem solvera a pomiarem bramki.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0500-gestszy-pomiar-odslania-dlug|20260801-0500-gestszy-pomiar-odslania-dlug]] - wspolne: iteracja, solver
<!-- /POWIAZANE:auto -->
