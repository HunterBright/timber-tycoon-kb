---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: pipeline/lessons
tags: [grafika, ikony, generatory-obrazow, comfyui, prompty, steam]
date: 2026-08-09
status: draft
---

# Ikona = jeden przedmiot. Scena to relacje, a modele obrazowe relacji nie trzymają

## Objaw

Kolejne próby ikony osiągnięcia „pierwsze ścięte drzewo" wracały z inną wadą za każdym razem:
siekiera wbita w pieniek zamiast w stojące drzewo, wióry wiszące w powietrzu, głowa siekiery
posklejana z dwóch kawałków, korona świerka jak płaskie pióra. Poprawianie opisu (dokładniejsza
anatomia, więcej kroków, wyższe CFG, zakazy w negatywie) podnosiło jakość **detalu**, ale nie
usuwało błędów **logicznych** - te wracały w innym miejscu.

## Przyczyna

Scena to nie suma obiektów, tylko obiekty **plus relacje między nimi**: co w czym tkwi, co pod
czym leży, co skąd wyrasta. Model obrazowy losuje prawdopodobny obraz - detal pojedynczego
przedmiotu ma opanowany, ale relacje wychodzą losowo. Przy każdym losowaniu któraś jest zła,
więc iteracja nie zbiega: naprawiasz jedną, psuje się druga.

## Rozwiązanie

**Jeden przedmiot na ikonę, zero sceny.** Nie „siekiera wbita w świerk z wiórami u stóp",
tylko „siekiera". Nie „taboret przed drzwiami domku", tylko „skrzynia". Bez relacji nie ma
czego pomylić - zostaje kształt, a z kształtem model radzi sobie za pierwszym podejściem.

Efekt uboczny: zestaw ikon staje się **bardziej spójny**, bo wszystkie są zbudowane tak samo
(przedmiot na jednolitym tle), zamiast być siedemnastoma różnymi scenkami.

Treść, której nie da się pokazać jednym przedmiotem, niesie **nazwa osiągnięcia** - i to
wystarcza, bo nazwa jest wyświetlana obok ikony.

## Analogia, która to tłumaczy

Przy postaciach 3D ten sam generator dawał świetne wyniki, bo odpowiadał **tylko za styl** -
proporcje i budowę pilnował skrypt Blendera (liczby, pomiary, bramki). Przy ikonach oddaliśmy
mu wszystko naraz: styl, budowę przedmiotów i logikę sceny. Zasada ogólna brzmi:

> Modelowi generatywnemu oddaj STYL. Poprawność musi pochodzić skądinąd -
> albo z innej warstwy (skrypt, pomiar), albo z uproszczenia zadania tak,
> żeby nie było czego pomylić.

## Wskazówki towarzyszące (sprawdzone przy tej samej robocie)

- **Rama i inne stałe elementy nie mogą być w opisie.** Rama opisana słowami wychodzi za każdym
  razem inna. Musi być plikiem, na który wkleja się generowane wnętrze.
- **Tło do wycięcia rób w kolorze spoza palety motywu** (magenta przy motywach drewno-las).
  Tło w kolorze bliskim motywowi jest nie do oddzielenia - maska zjada części przedmiotu.
- **Wycinaj tło rozlewaniem od krawędzi, nie samym progiem koloru.** Model dorzuca na tle
  poświaty i gradienty, które progiem koloru się nie łapią, a są ciągłe z tłem.
- **Generuj 4-6 kandydatów i wybieraj okiem.** Budowa narzędzi wychodzi losowo; jedna próba
  to loteria, sześć prób to wybór.
