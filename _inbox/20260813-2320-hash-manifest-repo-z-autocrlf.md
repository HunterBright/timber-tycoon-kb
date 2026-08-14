---
title: Manifest odciskow plikow w repo z autocrlf liczy sie po normalizacji koncow linii
type: lesson
status: draft
confidence: high
verified: '2026-08-13'
date: 2026-08-13
project: Another Quest
tags:
- git
- narzedzia
- bramki
- windows
applies_to: []
source: 'noc D1 sprintu AQ, job [S2b] - tools/canon_manifest.py'
severity: medium
suggested-category: workflow/lessons
time_lost: '~15 min (zlapane przy projektowaniu, nie po fakcie)'
---

# Manifest odciskow plikow w repo z autocrlf liczy sie po normalizacji koncow linii

## Problem

Bramka pilnujaca, czy ktos nie zmienil pliku poza legalna droga, liczy sha256 pliku i porownuje
z zapisanym odciskiem. Na Windowsie z `core.autocrlf=true` ten sam plik ma na dysku CRLF, a w indeksie
gita LF. Odcisk **surowych bajtow** zmienia sie przy samym `git checkout`, `git clone` albo zmianie
`.gitattributes` - bramka zaczyna zglaszac zmiany, ktorych nikt nie zrobil.

## Root cause

Odcisk bajtow odpowiada na pytanie "czy plik na dysku jest bajt w bajt taki sam", a pilnujemy czegos
innego: "czy TRESC dokumentu jest ta sama". Konce linii sa wlasnoscia checkoutu, nie tresci.

## Solution

Liczyc odcisk tekstu po normalizacji, nie bajtow:

```python
tekst = sciezka.read_bytes().decode("utf-8-sig", errors="replace")   # -sig zdejmuje BOM
tekst = tekst.replace("\r\n", "\n").replace("\r", "\n")
odcisk = hashlib.sha256(tekst.encode("utf-8")).hexdigest()
```

`utf-8-sig` jest rownie wazne: edytor, ktory raz zapisze plik z BOM-em, zmienia trzy bajty na poczatku
i wywraca odcisk tak samo jak konce linii.

## What didn't work

Rozwazane i odrzucone: wylaczenie autocrlf w repo. To leczy objaw w jednym repo i psuje pozostale
(inne narzedzia i edytory na tej maszynie zakladaja CRLF), a bramka i tak musi byc odporna, bo klon
na drugiej maszynie moze miec inne ustawienie.

## Generalization

Kazda bramka porownujaca "czy to sie zmienilo" musi najpierw nazwac, CO uznaje za tresc. Falszywy alarm
kosztuje tyle samo zaufania co przeoczenie - a bramka, ktora zapala sie po kazdym klonie, zostanie
wylaczona w tydzien.
