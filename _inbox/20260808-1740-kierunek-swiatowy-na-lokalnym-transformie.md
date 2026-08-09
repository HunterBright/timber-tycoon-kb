---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [unity, transform, przestrzen-lokalna, animacja, obiekt-rodzic, diagnoza]
date: 2026-08-08
status: draft
---

# Wektor światowy nałożony na lokalny transform - wada widoczna dopiero przy obróconym rodzicu

## Objaw

Ścięte drzewo przewracało się w jedną stronę, a leżący pień, który po okrzesaniu zastępuje
model drzewa, pojawiał się w innym miejscu - odsunięty o metr na drugą stronę pniaka.
Wyglądało to jak błąd spawnu pnia. Nie było.

## Przyczyna

Kierunek upadku losowany jest **w układzie świata**:

```csharp
Vector3 fallDir = new Vector3(Mathf.Cos(a), 0f, Mathf.Sin(a));   // świat
```

a animacja przewracania nakładała go na **lokalne** pola transformu:

```csharp
treeTransform.localRotation = Quaternion.AngleAxis(theta, fallAxis) * startRotation;
treeTransform.localPosition = elevatedStart + fallDir * slide;
```

Dopóki rodzic obiektu ma obrót zerowy, jedno równa się drugiemu i nic nie widać.
Drzewa stały pod obróconym rodzicem (losowy obrót, żeby nie wyglądały jak klony), więc
drzewo przewracało się w kierunku `obrótRodzica * fallDir`, a wszystko, co powstaje potem
(pień, kłody, kurz, sweep kolizji), liczyło się w świecie wzdłuż `fallDir`.

## Dlaczego trudno to złapać

1. **Wada jest niewidoczna przy rodzicu bez obrotu** - czyli w każdej prostej scenie testowej.
2. **Objaw wskazuje na złego winowajcę.** Widać "pień w złym miejscu", więc szuka się błędu
   w kodzie spawnu pnia. Kod spawnu pnia jest poprawny; błędna jest animacja.
3. **Rozmiar odchylenia jest losowy** - równy losowemu obrotowi konkretnego drzewa. Raz jest
   niemal niewidoczny, raz o 149 stopni. Wygląda to jak "czasem się psuje", a jest deterministyczne.

## Jak to rozstrzygnąć w minutę zamiast w godzinę

**Porównaj klatkę TUŻ PRZED podmianą modelu z klatką TUŻ PO.** W tym momencie kamera stoi
nieruchomo, więc przesunięcie widać wprost, bez mierzenia czegokolwiek. Nagranie z rozgrywki
wystarczy - nie trzeba odtwarzać sytuacji.

Potem **weź liczby z logu**, nie z oka: pozycja podstawy, pozycja spawnu, kierunek. Kąt między
"gdzie faktycznie wylądowało" a "gdzie miało wylądować" nazywa przyczynę: kąt bliski obrotowi
rodzica = pomylone przestrzenie.

## Reguła

Wektor policzony w świecie wolno nakładać wyłącznie na `position` / `rotation`.
Jeśli musi trafić na `localPosition` / `localRotation`, **przelicz go na układ rodzica**:

```csharp
Vector3 lokalnaOs   = rodzic.InverseTransformDirection(swiatowaOs).normalized;
Vector3 lokalnyRuch = rodzic.InverseTransformVector(swiatowyKierunek);  // nie Direction - niesie skalę
```

`InverseTransformDirection` do osi obrotu, `InverseTransformVector` do przesunięć - to drugie
uwzględnia skalę rodzica, więc metr w świecie zostaje metrem w świecie.

## Zostaw po sobie miernik

Naprawa dostała jednolinijkowe samosprawdzenie liczące kąt między faktycznym odsunięciem
a kierunkiem upadku, z werdyktem w logu i ostrzeżeniem powyżej progu. Koszt: jedna linijka.
Zysk: wada tej klasy nie wróci po cichu, bo każde użycie mechaniki ją mierzy.
