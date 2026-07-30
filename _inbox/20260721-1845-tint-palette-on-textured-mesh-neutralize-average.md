---
title: 'Paleta kolorow na TEKSTUROWANEJ siatce: dziel tint przez sredni kolor tekstury'
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-07-21'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- materialpropertyblock
- texture
- color-palette
- variants
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Paleta kolorow na TEKSTUROWANEJ siatce: dziel tint przez sredni kolor tekstury

## When to use
Masz JEDNA siatke i JEDEN material, a N wariantow rozniacych sie tylko kolorem (gatunki
drewna, frakcje, poziomy rzadkosci, warianty lakieru). Kolor wariantu jedzie przez
`MaterialPropertyBlock` (`_BaseColor`), zeby nie robic N materialow.

Wzorzec wchodzi w gre w momencie, gdy do dotad PLASKIEJ (bialej) siatki dokladasz
teksture - albo gdy wymieniasz teksture na inna o innej jasnosci.

## Steps
1. Zmierz sredni kolor tekstury (RGB, nie tylko luma). Np. skryptem po pikselach.
2. Wpisz go do configu jako pole `Color` z opisem, po co jest.
3. Kolor podawany do MPB licz jako `paleta[wariant] / sredniKoloruTekstury` (per kanal,
   z zabezpieczeniem `Max(0.01, x)` przed dzieleniem przez zero).
4. Dopisz check do build-smoke: dla KAZDEGO wariantu `tintMPB * sredniTekstury ~= paleta[wariant]`
   z tolerancja rzedu 0,02. Dzwignia porazki: wylacz dzielenie -> check pada.

## Why this works
URP/Lit mnozy `_BaseMap * _BaseColor`. Paleta jest zwykle autorowana pod BIALA powierzchnie,
wiec po nalozeniu tekstury o srednim kolorze `A` kazdy wariant wychodzi `paleta * A` - czyli
ciemniejszy i przesuniety w strone barwy tekstury. Dzielenie przez `A` sprawia, ze SREDNI
kolor wyrenderowanej powierzchni rowna sie dokladnie kolorowi z palety, a tekstura zostaje
czysta MODULACJA wokol tej sredniej.

Praktyczny zysk: dodanie tekstury przestaje byc zmiana designerska. Nikt nie musi
przestrajac palety, a zmiana jest weryfikowalna liczbowo, nie "na oko".

## Trade-offs
- `_BaseColor` moze wyjsc powyzej 1,0 dla jasnych wariantow (np. 0,85 / 0,80 = 1,06). Dla
  Lit/Unlit to bezpieczne, ale najjasniejsze piksele tekstury moga sie przepalic. Jesli
  tekstura ma mocne swiatla, warto ja najpierw sciemnic, zamiast podbijac tint.
- Wzorzec zaklada, ze tekstura jest w miare JEDNORODNA (drewno, tkanina, tynk). Dla tekstury
  z duzymi ciemnymi i jasnymi polami srednia niewiele mowi i efekt bedzie mylacy.
- MPB lamie SRP Batcher dla tego renderera. Przy pojedynczych obiektach bez znaczenia, przy
  setkach instancji trzeba isc w GPU instancing z per-instance properties.

## Variants
- **Jasnosc zamiast koloru**: dziel tylko przez lume tekstury, gdy chcesz zachowac barwe
  tekstury (np. drewno ma pozostac lekko bezowe niezaleznie od wariantu).
- **Bez neutralizacji**: ustaw sredni kolor na bialy. Zostawienie tego jako wartosci w configu
  (a nie flagi w kodzie) daje darmowy przelacznik do porownania obu wersji w buildzie.
- **Kolor kontrastowy z tej samej palety**: gdy na wariancie trzeba narysowac czytelny
  element (kreska, ikona, obrys), wybieraj jego kolor z LUMY wariantu i progu rownym
  sredniej jasnosci dwoch kolorow skrajnych - to maksymalizuje najgorszy przypadek.

## Related
- [[20260721-1830-linerenderer-flat-on-surface-invisible]]
