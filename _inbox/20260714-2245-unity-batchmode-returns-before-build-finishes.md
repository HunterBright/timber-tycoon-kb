---
type: lesson
project: Timber Tycoon
suggested-category: workflow/lessons
tags: [unity, build, batchmode, ci, verification, false-green]
date: 2026-07-14
status: draft
---

# Unity w trybie wsadowym WRACA, zanim build się skończy - i sonda daje fałszywe zielone światło

## Severity
Wysoka. Nie psuje kodu, ale **unieważnia bramkę jakości**: testujesz starą grę i dostajesz PASS.
To najgorszy możliwy rodzaj błędu w narzędziu weryfikującym - taki, który mówi "wszystko dobrze".

## Kontekst

Standardowa bramka buildu: zbuduj grę w trybie wsadowym, potem uruchom na niej sondę smoke.

```powershell
& "Unity.exe" -batchmode -quit -projectPath "..." -executeMethod Build.Windows -logFile build.log
echo $LASTEXITCODE          # <- oczekujesz 0

.\Builds\Win64\Game.exe -meshaudit
```

## Lekcja

**`Unity.exe` na Windows potrafi wrócić NATYCHMIAST, a build leci dalej w tle jako osobny proces.**
Polecenie kończy się, `$LASTEXITCODE` bywa **pusty** (nie 0 - PUSTY), a skrypt beztrosko leci dalej
i uruchamia sondę na **poprzednim** buildzie.

Sonda przechodzi. Wynik: **zielone światło na kodzie sprzed zmian.**

To jest szczególnie zdradliwe, bo:
- exit code niczego nie zdradza (jest pusty, nie błędny),
- log buildu ISTNIEJE i wygląda sensownie (bo build faktycznie trwa i pisze do niego),
- raport buildu na dysku mówi "Succeeded" - ale to raport z POPRZEDNIEGO przebiegu,
- liczba PASS-ów się zgadza z poprzednim razem, więc nic nie zgrzyta.

## Jak to złapałem

Sonda dała 56/56 PASS - dokładnie tyle, co poprzednio, mimo że dołożyłem 6 nowych sprawdzeń.
Nowa sekcja w raporcie **w ogóle nie istniała**. Gdyby nowych checków było 0, nigdy bym nie zauważył.

Znaczniki czasu rozstrzygnęły:
```
MeshAudit_Result.txt        16:20:57   <- sonda
Assembly-CSharp.dll         16:21:09   <- kod gry, 12 s PÓŹNIEJ
```
Sonda skończyła się, zanim build zdążył zapisać bibliotekę z kodem.

## Fałszywy trop po drodze

`Game.exe` miał datę sprzed dwóch dni i wyglądało to na "build w ogóle nie powstał".
**To normalne.** Plik `.exe` to tylko launcher - Unity go nie przepisuje, jeśli się nie zmienił.
**Kod gry siedzi w `Timber_Tycoon_Data/Managed/Assembly-CSharp.dll`** i to JEJ znacznik czasu jest
dowodem, że build jest świeży. Data pliku `.exe` nie znaczy nic.

## Naprawa

**Czekaj na PROCES, nie na polecenie.**

```powershell
& "Unity.exe" -batchmode -quit -projectPath "..." -executeMethod Build.Windows -logFile build.log

$deadline = (Get-Date).AddMinutes(12)
while ((Get-Date) -lt $deadline -and (Get-Process Unity -ErrorAction SilentlyContinue)) {
    Start-Sleep -Seconds 5
}
```

Dodatkowo, trzy zasady, które zamieniają "ufam" na "wiem":

1. **Kasuj artefakt przed przebiegiem.** Usuń raport buildu i raport sondy PRZED startem. Jeśli po
   przebiegu pliku nie ma - build nie doszedł do końca. Nie da się wtedy pomylić starego z nowym.
2. **Sondę uruchamiaj przez `Start-Process -Wait -PassThru`** i czytaj `$p.ExitCode`. To działa,
   gdy `$LASTEXITCODE` kłamie.
3. **Licz sprawdzenia.** Jeśli dodałeś 6 checków, a suma się nie zmieniła - sonda NIE uruchomiła
   twojego kodu. Suma PASS-ów jest darmowym wykrywaczem nieświeżego builda.

## Powiązane

To jest dokładnie ta sama rodzina, co [[probe-must-be-able-to-fail]]: narzędzie weryfikujące, które
nie umie zawieść (albo weryfikuje nie to, co myślisz), jest gorsze niż jego brak - bo produkuje
fałszywe poczucie bezpieczeństwa.
