---
title: "Zapisy w chmurze na Steamie: Auto-Cloud zamiast przepisywania systemu zapisu"
type: lesson
status: draft
confidence: medium
verified: ''
date: 2026-08-15
project: Kerf - Sawmill Tycoon
tags: [steam, unity, zapisy, chmura, wydanie]
applies_to: []
source: ''
severity: high
suggested-category: engine/lessons
time_lost: ''
---

# Zapisy w chmurze na Steamie: Auto-Cloud zamiast przepisywania systemu zapisu

## Lekcja

Gra jednoosobowa, która zapisuje się do plików, **nie potrzebuje żadnego kodu Steama**,
żeby mieć zapisy w chmurze. Wystarczy **Steam Auto-Cloud**: w panelu wskazujesz katalog
i wzorzec plików, Steam kopiuje je przy wyjściu z gry i przywraca przy starcie.
Wariant przez API (`ISteamRemoteStorage`) oznacza przepisanie systemu zapisu - czyli
ruszanie najbardziej wrażliwej części gry tuż przed premierą, bez zysku dla gracza.

## Konfiguracja dla Unity (Windows + macOS)

Unity zapisuje do `Application.persistentDataPath`:

- Windows: root `WinAppDataLocalLow`, podkatalog `<firma>/<nazwa gry>/...`
- macOS: root `MacAppSupport`, podkatalog `<firma>/<nazwa gry>/...`

Trzeba dodać **osobny wiersz na każdy system** (pole Operating System).
Ustawienie działa dopiero po **opublikowaniu** w panelu - samo "Save" nic nie daje.

## Mina: zmiana nazwy gry po cichu wyłącza chmurę

Ścieżka zawiera `companyName` i `productName` z ProjectSettings. Zmiana nazwy gry
(branding, podtytuł, myślnik) przenosi katalog zapisów w inne miejsce, a ustawienie
chmury dalej pilnuje starego - **chmura przestaje działać bez jednego komunikatu błędu**.
Nazwę gry trzeba zamrozić przed konfiguracją chmury; jeśli musi się zmienić, zmienia się
też ustawienie w panelu ORAZ dopisuje migrację katalogu zapisów w kodzie.

## Co świadomie zostawić poza chmurą

- ustawienia ekranu (rozdzielczość, jakość) - to cecha KOMPUTERA, nie gracza;
  zsynchronizowane potrafią odpalić słabszą maszynę w trybie, którego nie uciągnie
- dzienniki i telemetrię - rosną bez końca, nikomu nie są potrzebne na drugim komputerze
- własne kopie zapasowe zapisów - wzorzec plików trzeba dobrać tak, żeby ich nie łapał

## Powiązane

- [[20260815-1200-klucze-steam-przed-premiera|Zwykłe klucze Steam wysłane przed premierą = twórca nie zagra]] - wspolne: steam, wydanie
