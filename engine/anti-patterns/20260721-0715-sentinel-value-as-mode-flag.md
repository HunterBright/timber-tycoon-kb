---
title: 'Sentinel value: wnioskowanie TRYBU z mierzonej liczby zamiast jawnej flagi'
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-21'
project: Kerf - Sawmill Tycoon
tags:
- unity
- state-machine
- floating-point
- animation
- vehicles
- code-smell
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Sentinel value: wnioskowanie TRYBU z mierzonej liczby zamiast jawnej flagi

## The trap

Masz dwa źródła jednej wartości (np. kąt skrętu koła: raz liczy go manewr skryptowany, raz
sterowanie gracza/AI). Kusi, żeby nie dokładać osobnej flagi i wywnioskować tryb z samych
danych, bo "przecież gdy manewr nie trwa, ta liczba i tak jest zerem":

```csharp
bool usingOverride = speedOverride != 0f;              // <-- pułapka
float angle = usingOverride ? steerOverride : inputAngle;
```

Wygląda oszczędnie: zero dodatkowego stanu, jedno źródło prawdy, brak ryzyka rozjazdu flagi
z rzeczywistością. To dokładnie odwrotność prawdy.

## Why it fails

**Wartość zerowa jest LEGALNYM odczytem trybu włączonego, nie tylko sygnałem trybu wyłączonego.**
Mierzona wielkość fizyczna (prędkość, przesunięcie, delta) przechodzi przez zero za każdym razem,
gdy ruch zwalnia, startuje, kończy się lub gdy dzielnik wpadnie w bezpiecznik. Każde takie zero
przełącza tryb na jedną klatkę i renderuje wartość z DRUGIEJ gałęzi w pełnej amplitudzie.

Gorzej: taki test to `==` na floatach przebrane za logikę biznesową. Nie ma progu, nie ma
histerezy, nie ma nazwy - więc nikt go nie czyta jak decyzji o trybie i nikt go nie testuje.

Konkret, który to wywołał: prędkość manewru liczono jako
`Time.deltaTime > 0.001f ? droga/dt : 0f`. Build chodził ~886 FPS, więc **92,9% klatek miało
dt <= 0,001 s** i dostawało twarde zero. Przez 93% klatek koła uznawały, że manewr nie trwa,
i rysowały kąt z drugiej gałęzi (0 stopni). Efekt na ekranie: koła migoczące między prawidłowym
skrętem a pozycją na wprost, tym mocniej, im większy kąt.

Ten sam projekt naprawiał tydzień wcześniej bliźniaczy błąd: `Mathf.Sign()` z rzutu prędkości,
który przy parkowaniu bokiem skakał +/- co klatkę. Antywzorzec wraca, bo za każdym razem wygląda
jak oszczędność, a nie jak decyzja.

## Symptoms

- Element wizualny migocze między poprawną wartością a wartością "domyślną/zerową", zamiast
  między dwiema poprawnymi.
- Amplituda migotania rośnie wraz z wartością sygnału ("im ostrzejszy zakręt, tym gwałtowniej").
- Objaw zależy od klatkażu: nie odtwarza się w Edytorze przy 60 FPS, wychodzi w buildzie.
- W kodzie: `!= 0f`, `> 0f`, `Mathf.Sign(...)`, `Mathf.Approximately(x, 0)` użyte jako **warunek
  wyboru gałęzi**, a nie jako zabezpieczenie obliczenia.
- Filtr wygładzający istnieje, ale siedzi PRZED tym testem - więc wygładza jedną gałąź, a nie to,
  co faktycznie trafia na wyjście. Analiza "przecież to jest filtrowane" jest wtedy fałszywa.

## Correct approach

1. **Jawna flaga trybu, ustawiana przez właściciela manewru.** Tryb to fakt o systemie, nie
   wniosek z pomiaru:
   ```csharp
   [System.NonSerialized] public bool OverrideActive;   // ustawia manewr, na start i na koniec
   bool usingOverride = OverrideActive;
   ```
   Koszt: dwa przypisania. Zysk: tryb przestaje zależeć od tego, jak szybko chodzi gra.
   Uwaga na dyscyplinę: **oba** przypisania muszą wejść, a ścieżka przerwania manewru też musi
   flagę gasić.

2. **Ogranicznik zmiany w JEDYNYM miejscu, które pisze wyjście.** Niezależnie od źródła:
   ```csharp
   rendered = Mathf.MoveTowards(rendered, target, maxUnitsPerSecond * Time.deltaTime);
   ```
   To zamyka całą klasę błędu: żadne przełączenie gałęzi, zerowanie ani skok martwej strefy nie
   ma jak narysować się jako przeskok. Filtr MUSI być za multiplekserem, nie przed nim.

3. **Jeśli naprawdę musisz wnioskować z danych** - użyj progu z histerezą i nazwij go, zamiast
   porównywać do zera.

4. **Test regresji mierzy RÓŻNICĘ MIĘDZY KOLEJNYMI KLATKAMI**, w jednostkach na sekundę (nie na
   klatkę - inaczej próg zależy od FPS), odczytaną z narysowanego stanu. Testy mierzące maksimum
   lub wartość chwilową są na ten błąd ślepe z definicji: migoczący element ma takie samo
   maksimum jak zdrowy.

## Related

- [[20260721-0725-frame-time-epsilon-guard-breaks-at-high-fps]] - drugi mechanizm tej samej awarii
- [[build-is-the-only-truth-editor-lies]]
