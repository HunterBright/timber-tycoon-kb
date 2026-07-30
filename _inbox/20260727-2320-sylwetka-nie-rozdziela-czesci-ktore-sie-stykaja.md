---
title: Sylwetka nie rozdziela dwóch rzeczy, które się stykają — i milczy o tym
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-27'
project: Kerf - Sawmill Tycoon
tags:
- pomiar
- sylwetka
- obraz
- referencje
- modelowanie
- blender
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Sylwetka nie rozdziela dwóch rzeczy, które się stykają — i milczy o tym

## Objaw

Głowa budowana z pomiaru sylwetki wychodziła jak beczka. Wszystkie pomiary
były poprawne: szerokość z rzutu z przodu, głębokość z rzutu z boku, wysokości
z załamań obrysu. A jednak proporcje były bez sensu — głowa niemal tak szeroka
jak głęboka.

## Przyczyna

**Uszy.** W rzucie z przodu ucho i czaszka tworzą jedną plamę, więc „szerokość
głowy na tej wysokości" znaczy w rzeczywistości „od ucha do ucha": 19,2 cm
zamiast 15,9 cm. W rzucie z boku ucho leży całkowicie WEWNĄTRZ obrysu głowy,
więc drugi rzut też o nim nie mówi.

Żaden pomiar obrysu nie potrafi tego rozdzielić, bo dla obrysu to jeden
obiekt. I — co gorsze — pomiar nie sygnalizuje problemu: oddaje sensownie
wyglądającą liczbę.

## Rozwiązanie

Granica między czaszką a uchem jest w referencji NARYSOWANA jako krawędź
siatki. Wystarczyło wziąć wykrywacz krawędzi używany do czegoś innego
(poziome pętle) i przestawić go na kreski PIONOWE, a potem zapytać:
na jakiej odległości od osi jest ich najwięcej w paśmie uszu.

Kreski twarzy są rozrzucone, a linia odejścia ucha powtarza się na całej
jego wysokości, więc wystaje ponad tło jak pik w histogramie. Wynik: 15,9 cm,
zgodny z tym, co widać na obrazku.

Ważny szczegół: funkcja **oddaje None**, gdy pik nie jest co najmniej dwa razy
wyższy od tła, a wtedy model wraca do szerokości sylwetki i wypisuje ostrzeżenie.
Lepiej beczka z komunikatem niż cicho zmyślona liczba.

## Reguła ogólna

Zanim uznasz obrys za pomiar części ciała, sprawdź, **czy do tej części nie
przylega coś innego** — ucho do czaszki, ręka do tułowia, palec do dłoni,
skrzynia do burty. Jeśli przylega, obrys mierzy sumę i nie ma jak tego wykryć
po samym wyniku.

Test praktyczny: policz stosunek szerokości do głębokości i porównaj
z oczekiwanym. Rozjazd rzędu 20% zwykle znaczy, że do sylwetki wchodzi coś,
o czym zapomniałeś.

Bardziej ogólnie: przecięcie rzutów (visual hull) i pomiar obrysu widzą tylko
to, co zmienia kontur. Wgłębienia (oczodół, usta, szew między palcami) i
elementy przylegające (ucho) są dla nich niewidzialne. Do jednych i drugich
potrzeba innego źródła — u nas: narysowanych krawędzi.
