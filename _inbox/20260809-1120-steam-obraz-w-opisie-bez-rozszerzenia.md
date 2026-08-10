---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: publishing/lessons
tags: [steam, steamworks, strona-sklepu, opis, obrazy, bbcode]
date: 2026-08-09
status: draft
---

# Obraz w opisie na Steamie: etykieta BEZ rozszerzenia, nie nazwa pliku

## Objaw

Animacje wstawione do sekcji „O tej grze" nie wyświetlały się - w miejscu obrazu pusty
prostokąt. Znaczniki wyglądały poprawnie, pliki były wgrane, nazwy pod miniaturami się
zgadzały.

## Jak działa naprawdę

Steam **nie przechowuje wgranego pliku pod jego nazwą**. Plik `1_scinka_drzewa.mp4` ląduje
na serwerze jako `54ac5d6ec113f6765e32c8029d91ce5e.webm` plus wygenerowany podgląd
`.poster.avif`. Nazwa widoczna pod miniaturą w panelu to **etykieta**, nie nazwa pliku.

W znaczniku podaje się **etykietę BEZ rozszerzenia**:

```
[img]{STEAM_APP_IMAGE}/extras/1_scinka_drzewa[/img]     <- działa
[img]{STEAM_APP_IMAGE}/extras/1_scinka_drzewa.gif[/img] <- pusty prostokąt
[img]{STEAM_APP_IMAGE}/extras/1_scinka_drzewa.mp4[/img] <- pusty prostokąt
```

Steam sam dobiera właściwy wariant zasobu do przeglądarki. Podanie konkretnego hasha
z rozszerzeniem też zadziała, ale jest **kruche**: przy ponownym wgraniu tego samego
materiału hash się zmienia i znacznik umiera. Etykieta przeżywa podmianę.

## Czego nie robić

Nie zgadywać rozszerzenia i nie iść w hash tylko dlatego, że jest „bardziej konkretny".
Straciłem na tym kilka podejść: najpierw `.gif` (bo takie były pliki na dysku), potem
wyciąganie hashy z DOM-u panelu i wstawianie ich z `.webm`. Obie drogi dają wynik, który
albo nie działa, albo działa do pierwszej podmiany zasobu.

## Przy okazji: formaty i szerokości

- Sekcja opisu ma **780 px** szerokości na komputerze (nie 616 - to szerokość innej kolumny).
- W opisie działają **MP4 i WEBM**, nie tylko GIF. Steam i tak konwertuje do WEBM.
  Przy tej samej długości MP4 daje kilkukrotnie wyższą rozdzielczość niż GIF z paletą 256 kolorów.
- Animacja może trwać maksymalnie 12 sekund.

## Reguła ogólna

Gdy platforma pokazuje nazwę zasobu w panelu, sprawdź, czy to nazwa pliku, czy identyfikator.
Wstawiając odwołanie, użyj tego, co daje przycisk „kopiuj nazwę" platformy - nie tego,
co widzisz na dysku.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260809-1620-steam-strona-sklepu-edycja-programowa|Steam - co da się zmienić na stronie sklepu programowo, a co się zatrzaskuje po publikacji]] - wspolne: steamworks, steam
- [[20260724-1907-steam-release-timeline-two-reviews-hard-clock|Wydanie gry na Steam: sekwencyjne recenzje + twardy zegar Coming Soon]] - wspolne: steamworks, steam
<!-- /POWIAZANE:auto -->
