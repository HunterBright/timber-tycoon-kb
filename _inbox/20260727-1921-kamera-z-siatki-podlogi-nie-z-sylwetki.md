---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [fotogrametria, kamera, punkty-zbiegu, blender, referencje, pomiar]
date: 2026-07-27
status: draft
---

# Kamerę odtwarzaj z regularnej struktury sceny, nie z sylwetki modelu

## Objaw

Mając sześć renderów tego samego modelu z różnych stron, chciałem odtworzyć
ustawienia kamer, żeby wyrzeźbić bryłę z przecięcia obrysów. Dopasowywanie
przez obracanie bryły próbnej, aż jej obrys pokryje się z obrysem ze zdjęcia,
dawało wyniki bezsensowne: dla ujęcia oczywiście poziomego wychodziło 31 stopni
w górę, a symetryczna postać myliła przód z tyłem w czterech ujęciach na sześć.

## Przyczyna

**Błąd kształtu bryły próbnej był większy niż różnica między kamerami.**
Solver nadrabiał niedokładność modelu kątem kamery. To jest ogólna pułapka:
dopasowanie „model do obrazu" działa tylko wtedy, gdy model jest już dobry —
a jeśli model dopiero budujesz, to jest błędne koło.

## Rozwiązanie

Odtwarzaj kamerę z czegoś, co **nie zależy od modelu**. W renderze z widoku
Blendera są dwie takie rzeczy:

1. **Kolorowe osie świata** (czerwona X, zielona Y) — dają KIERUNEK patrzenia.
2. **Siatka podłogi** — daje POCHYLENIE i OGNISKOWĄ.

Kluczowe rozróżnienie, którego brakowało na początku: obraz jednej prostej mówi,
w którą stronę ona biegnie, ale **nie mówi, gdzie na niej leży punkt
nieskończenie daleki**. A to właśnie ten punkt (punkt zbiegu) niesie pochylenie
kamery. Do jego znalezienia potrzebne są DWIE proste równoległe — jedna oś ich
nie da, siatka podłogi daje kilkadziesiąt.

Rachunek:
- linie siatki dzielą się na dwa pęki, każdy zbiega w swoim punkcie,
- prosta przez oba punkty to horyzont,
- `tg(pochylenie) = (środek_kadru_y − horyzont_y) / ogniskowa`,
- prostopadłość osi X i Y daje ogniskową: `f² = −(v1−c)·(v2−c)`.

## Trzy rzeczy, które kosztowały osobne przebiegi

**1. Zgodność prostej z punktem zbiegu mierz W PIKSELACH, nie kątem.**
Przy punkcie zbiegu oddalonym o kilka tysięcy pikseli proste rozbiegające się
o kilometry mieszczą się w ułamku stopnia — kąt przestaje cokolwiek znaczyć.
Właściwa miara: o ile pikseli prosta mija punkt zbiegu, licząc na końcu
własnego odcinka (`długość/2 · sin(kąt)`).

**2. Szukanie musi karać rozwiązanie z jednym wielkim pękiem.** Przy horyzoncie
uciekającym daleko poza kadr wszystkie proste zlewają się w jeden pęk
i „zgodność" wychodzi wtedy największa z możliwych — szukanie samo się w to
pakuje. Lekarstwo: ILOCZYN siły dwóch pęków zamiast sumy, przy czym drugi pęk
liczy tylko te proste, których nie wziął pierwszy.

**3. Pęk zbiegający w nieskończoności wypada z układu W CAŁOŚCI.** Przy kamerze
na wprost jedna rodzina linii jest w kadrze równoległa. Zostawienie jej prostych
w układzie równań (bez kolumny na punkt zbiegu) sprawia, że każda z nich żąda
„horyzont leży na MOIM wierszu" — a prosta równoległa do horyzontu nie ma z nim
nic wspólnego.

## Jak udowodnić, że wyszło dobrze

To jest najcenniejsza część i warto ją powtarzać na innych projektach:
**przepuść tę samą strukturę z powrotem przez znalezioną kamerę.**

Rzuciłem znalezione linie siatki przez każdą kamerę na płaszczyznę podłogi
i zmierzyłem, co ile metrów wypadają. Wszystkie 12 rodzin z 6 ujęć dało
0,239–0,244 m (rozrzut 2%). Kontrola ma zęby: pochylenie zmienione o 2 stopnie
psuje rozstaw o 7–31%.

Dodatkowo, niezależnie:
- ogniskowa policzona osobno z czterech ujęć: rozrzut 0,4%, i wypadła dokładnie
  na domyślny obiektyw 50 mm Blendera (39,6 stopnia pola widzenia),
- środek rzutu wyszedł dokładnie na środku kadru — to nie było założenie,
  tylko wniosek z tej zgodności.

## Granica metody (i co z nią zrobić)

Gdy kamera patrzy niemal wzdłuż podłogi (mniej niż ~4 stopnie nad nią), cała
widoczna podłoga mieści się w wąskim pasie i sąsiednie linie siatki są bliżej
siebie niż dwa piksele. Dopasowanie prostej wciąga wtedy piksele sąsiadów
i wynik się wygina — o kilka stopni.

Nie da się tego obejść i **narzędzie ma wtedy ODMÓWIĆ, nie zgadywać**.
Rozpoznanie: powtarzaj odczyt, odcinając za każdym razem pas wokół
znalezionego horyzontu; jeśli po kilku przebiegach horyzont nadal skacze,
przerwij z jasnym komunikatem. Cicha zła liczba jest dużo gorsza niż odmowa.

## Kiedy to zastosować gdzie indziej

Wszędzie, gdzie trzeba odtworzyć kamerę ze zrzutów/renderów, a scena zawiera
cokolwiek regularnego: siatkę podłogi, kafle, cegły, deski, szyny, słupki
ogrodzenia. Struktura sceny bije dopasowywanie modelu — zawsze.
