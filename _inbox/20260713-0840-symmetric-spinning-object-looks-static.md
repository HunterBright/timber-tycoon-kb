---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, vfx, animation, perception, aliasing, game-feel]
date: 2026-07-13
status: draft
severity: medium
---

# Obracający się obiekt symetryczny wygląda na nieruchomy

## Objaw

Reżyser zgłosił: "tarcza piły jest statyczna, ma się szybko obracać". W kodzie obrót **był
od zawsze** i działał poprawnie (540 stopni/s, aplikowany co klatkę). Kusi, żeby "dodać obrót",
który już jest — i nic się nie zmieni.

## Przyczyna źródłowa (trzy nakładające się powody)

1. **Symetria obrotowa = brak informacji o obrocie.** Tarcza była gładkim krążkiem z 28
   identycznymi ząbkami. Obiekt, który w KAŻDYM położeniu kątowym wygląda tak samo, nie niesie
   żadnej informacji o tym, że się obraca. Oko nie ma czego śledzić. To nie kwestia prędkości —
   przy nieskończonej prędkości też wyglądałby na stojący.

2. **Aliasing stroboskopowy ("koło w westernie").** 28 ząbków = wzór powtarzalny co
   `360/28 = 12,86°`. Przy 540°/s i 60 FPS obrót między klatkami to `9°`. Ponieważ 9° jest
   BLISKO 12,86°, oko interpretuje ruch jako **powolne cofanie się** (9 − 12,86 = −3,86°/klatkę)
   albo jako drganie w miejscu. Klasyczny efekt wozu w westernie.

3. **Skrót perspektywiczny.** Kamera patrzyła na tarczę pod ~21° od jej płaszczyzny, więc widać
   było mocno spłaszczoną elipsę — jeszcze mniej materiału do śledzenia.

## Naprawa

Nie „dodaj obrót" (był), tylko **daj oku punkt zaczepienia**: złam symetrię cechą o NISKIEJ
częstotliwości kątowej.

- ciemna piasta w środku + **3 jasne nity co 120°**
- wzór o okresie 120° przy kilkuset °/s czyta się jednoznacznie jako szybki obrót i **nie
  aliasuje się** z klatkami (120° >> 15°/klatkę)
- kontrast materiału (mosiądz na stali) zamiast jednolitego koloru

## Reguła projektowa

Przy każdym szybko obracającym się elemencie (tarcze, koła, wentylatory, bębny) sprawdź:

1. Czy obiekt ma **cechę asymetryczną**, którą oko może śledzić? Jeśli nie — dodaj (nit, rysa,
   wycięcie, malowany znak, piasta).
2. Jaki jest **okres kątowy** powtarzalnego wzoru (N zębów -> 360/N)? Obrót na klatkę
   (`speed / FPS`) musi być **wyraźnie mniejszy** od tego okresu, inaczej dostajesz
   stroboskop. Bezpiecznie: obrót/klatkę < 1/4 okresu wzoru.
3. Czy kamera widzi obiekt **z twarzy**, czy niemal krawędzią? Skrót perspektywiczny zjada
   czytelność ruchu.

## Sygnał ostrzegawczy do zapamiętania

Gdy ktoś mówi **"to jest nieruchome"** o czymś, co w kodzie ewidentnie się rusza — problem nie
leży w transformacji, tylko w **percepcji**. Nie debuguj obrotu, debuguj to, co widać.
