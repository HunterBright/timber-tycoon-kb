---
type: anti-pattern
project: Kerf - Sawmill Tycoon
suggested-category: process/anti-patterns
tags: [testy, bramki, sonda, efekty-uboczne, steam, osiagniecia]
date: 2026-08-09
status: draft
---

# Bramka nie może przyznawać tego, co ma sprawdzać

## Co zrobiłem źle

Sprawdzenie w sondzie buildowej miało dowieść, że odblokowanie osiągnięcia nie wywala gry,
gdy Steam nie działa. Wywoływało więc odblokowanie na **prawdziwym identyfikatorze**:

```csharp
SteamAchievements.Unlock(AchievementIds.FirstCut);   // ŹLE
```

Założenie brzmiało: sonda chodzi na gołym buildzie, Steama nie ma, więc wywołanie pójdzie
w próżnię. Założenie było fałszywe - Steam u autora był uruchomiony, warstwa się połączyła
i osiągnięcie **realnie wskoczyło na konto**. Bramka przyznała nagrodę zamiast ją sprawdzić.

## Dlaczego to groźniejsze niż wygląda

1. **Fałszywy dowód.** „Osiągnięcie się odblokowało" przestaje znaczyć „gracz to zrobił".
   Gdyby to weszło do wydania, każde uruchomienie sondy skażałoby konto testowe.
2. **Nieodwracalność.** Odblokowania na Steamie kasuje się tylko hurtem, razem
   z prawdziwymi zdobyczami gracza.
3. **Objaw wygląda jak sukces.** W dzienniku stoi `odblokowane: ACH_FIRST_CUT` i łatwo
   uznać to za dobrą wiadomość, a nie za skutek uboczny bramki.

## Zasada

**Sprawdzenie wolno prowadzić wyłącznie po danych, których skutek nie wychodzi poza sondę.**
Gdy testowana ścieżka kodu ma efekt zewnętrzny (konto gracza, zapis, zakup, wysyłka),
trzeba ją wywołać na danych zawsze odrzucanych - identyfikatorze, którego nie ma
w konfiguracji, pustym ciągu, wartości pustej. Ścieżka kodu jest ta sama, więc test nic
nie traci, a skutek uboczny znika.

```csharp
SteamAchievements.Unlock(null);                    // DOBRZE
SteamAchievements.Unlock("");
SteamAchievements.Unlock("ACH_NIE_ISTNIEJE_TAKIE");
```

## Pytanie kontrolne przed dopisaniem sprawdzenia do bramki

> Gdyby to sprawdzenie zadziałało **w pełni poprawnie** na komputerze użytkownika,
> co zostanie zmienione poza plikiem z wynikiem?

Jeśli odpowiedź nie brzmi „nic", sprawdzenie trzeba przepisać.

Powiązane: bramka musi mieć udowodniony tryb porażki (dźwignia) - to osobna zasada,
która TEJ pomyłki nie wyłapuje. Dźwignia dowodzi, że check umie oblać; nie mówi nic
o tym, co check psuje po drodze.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260808-1145-slot-roboczy-sondy-zjada-dostarczany-artefakt|20260808-1145-slot-roboczy-sondy-zjada-dostarczany-artefakt]] - wspolne: sonda, testy
- [[20260731-2200-slepa-dzwignia-debugger-bramek|20260731-2200-slepa-dzwignia-debugger-bramek]] - wspolne: sonda, bramki
- [[20260731-0930-bramka-mierzaca-srodek-bryly-oblewa-ksztalty-niesymetryczne|Sprawdzian celujący w środek bryły oblewa kształty, których masa jest przesunięta]] - wspolne: testy, bramki
- [[20260726-1930-zielone-bramki-nie-dowodza-ze-wyglada-dobrze|Zielona tablica bramek nie dowodzi, ze cos wyglada dobrze]] - wspolne: testy, bramki
- [[20260720-0920-red-proof-musi-uzbroic-wszystkie-checki|Dźwignia psująca jeden check może wyłączyć drugi - i jego tryb porażki zostaje nieudowodniony]] - wspolne: sonda, testy
- [[gate-must-have-provable-failure-mode|Bramka bez udowodnionego trybu porazki niczego nie pilnuje]] - wspolne: sonda, testy
<!-- /POWIAZANE:auto -->
