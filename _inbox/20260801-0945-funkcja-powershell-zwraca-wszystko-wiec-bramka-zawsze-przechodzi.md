---
title: Funkcja PowerShella zwraca WSZYSTKO, wiec sprawdzenie jej wyniku zawsze wychodzi na prawde
type: anti-pattern
status: draft
confidence: high
verified: 2026-08-01
tags: [powershell, bramki, automatyzacja, cichy-blad, skrypty, false-green]
date: 2026-08-01
project: GameDevOS
suggested-category: workflow/general
source: bramka bezpieczenstwa przed wysylka na GitHuba, 2026-08-01
applies_to: [kazdy skrypt PowerShell, ktory podejmuje decyzje na podstawie wyniku wlasnej funkcji]
severity: high
time_lost: ok. 15 min, ale blad przepuscil zatrzymana wysylke jako sukces
---

# Funkcja PowerShella zwraca WSZYSTKO, wiec sprawdzenie jej wyniku zawsze wychodzi na prawde

## Antywzorzec

    function Zapisz-Repo {
        ...
        & python bramka.py --repo $sciezka        # <-- to pisze do potoku
        if ($LASTEXITCODE -ne 0) {
            Write-Host "BRAMKA ZATRZYMALA WYSYLKE"
            return $false
        }
        ...
        return $true
    }

    $ok = Zapisz-Repo -Sciezka $repo
    if ($ok) { Write-Host "PUBLIKACJA: OK" }

Wyglada poprawnie. Bramka zadzialala, wypisala „ZATRZYMALA WYSYLKE",
zwrocila `$false` - **i skrypt napisal „PUBLIKACJA: OK"**.

## Dlaczego

W PowerShellu wartoscia zwracana przez funkcje jest **wszystko, co trafilo
do potoku wyjsciowego**, a nie to, co stoi po slowie `return`. Wywolanie
zewnetrznego programu wypisuje swoje wiersze do potoku, wiec funkcja zwrocila
tablice: `@("BRAMKA: ZATRZYMANE...", "  plik:12 [klucz]...", $false)`.

Niepusta tablica jest w warunku **prawda**. Bramka zadzialala idealnie
i nie zmienila niczego, bo jej werdykt utonal we wlasnym wydruku.

To jest najgorszy rodzaj bledu w bramce: **bramka dziala, a mimo to przepuszcza**.
Log pokazuje, ze zlapala. Wynik pokazuje sukces. Nikt nie czyta obu naraz.

## Naprawa

Trzy rzeczy razem, bo kazda z osobna zostawia dziure:

1. Wypchnij wydruk poza potok: `& python bramka.py ... | Out-Host`
   (albo `[void](...)`, albo `| Out-Null`).
2. Bierz **ostatni** element wyniku: `$ok = @(Zapisz-Repo ...)[-1]`.
3. Porownuj **jawnie z wartoscia logiczna**: `if ($ok -eq $true)`, nie `if ($ok)`.
   Samo `if ($ok)` uzna za prawde kazdy niepusty napis.

## Bramka na te bramke

Zepsuj ja celowo raz i sprawdz, czy skrypt konczy sie **bledem**, a nie sukcesem.
Tu wystarczylo podlozyc plik z falszywym kluczem i zobaczyc, jaki komunikat
koncowy pada. Bez tego kroku blad zylby do pierwszego prawdziwego wycieku.

## Powiazane

- [[gate-must-have-provable-failure-mode]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260726-1415-powershell-nie-czeka-na-unity-batchmode|PowerShell nie czeka na Unity.exe ani na exe gry - kontrola swiezosci builda strzela za wczesnie]] - wspolne: powershell, false-green
<!-- /POWIAZANE:auto -->
