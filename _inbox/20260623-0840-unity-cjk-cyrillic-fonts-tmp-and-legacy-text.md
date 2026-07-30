---
title: Renderowanie CJK + cyrylicy w grze Unity (TextMeshPro + legacy UI.Text)
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-06-23'
project: Kerf - Sawmill Tycoon
tags:
- unity
- localization
- fonts
- textmeshpro
- cjk
- cyrillic
- ui-text
- noto
- i18n
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Renderowanie CJK + cyrylicy w grze Unity (TextMeshPro + legacy UI.Text)

## When to use
Gra ma już warstwę tłumaczeń (JSON/SO), ale po dodaniu języków rosyjskiego/
japońskiego/koreańskiego/chińskiego tekst pokazuje puste kwadraciki ("tofu").
Przyczyna prawie zawsze: w projekcie jest TYLKO czcionka łacińska (np. Nunito,
Roboto) bez glifów cyrylicy ani CJK. Tłumaczenia są poprawne — brakuje czcionki.
Częsty przypadek mieszany: część UI to nowy TextMeshPro, część to stary
UnityEngine.UI.Text — każdy wymaga INNEGO podejścia.

## Steps
1. **Jedna czcionka pokrywająca wszystko:** Noto Sans CJK (regionalny OTF, np.
   `NotoSansCJKsc-Regular.otf`, ~16MB). JEDEN regionalny plik zawiera PEŁNY
   repertuar: kana + Hangul + Han + łacina ASCII + cyrylica + Latin-1. Region
   (sc/jp/kr/tc) ustala tylko domyślny kształt Han. Źródło: GitHub
   `notofonts/noto-cjk` release (asset `08_NotoSansCJKsc.zip` → wyciągnij sam
   Regular OTF), licencja OFL (wolno dołączać do builda).
   - GOTCHA: ten font NIE ma Latin Extended-A (ł, č, ő, ş). Nieistotne, jeśli
     języki łacińskie zostają na swojej czcionce — patrz krok 3.
2. **TextMeshPro (większość UI) — fallback, ZERO kodu per język:**
   z OTF zrób TMP_FontAsset w trybie **Dynamic** (`TMP_FontAsset.CreateFontAsset(
   font, 90, 9, GlyphRenderMode.SDFAA, 1024, 1024, AtlasPopulationMode.Dynamic,
   true)`; pamiętaj `AddObjectToAsset` na atlas + materiał). Dopisz go do
   **globalnego** fallbacku w `TMP Settings` (`m_fallbackFontAssets` przez
   SerializedObject) i/lub do `fallbackFontAssetTable` głównej czcionki. TMP sam
   pobiera brakujący glif z fallbacku — cyrylica i CJK działają wszędzie bez
   logiki per język. Dynamic = rasteryzuje tylko użyte znaki (atlas mały).
3. **Legacy UnityEngine.UI.Text — czcionka zależna od języka:** stary Text NIE
   ma łańcucha fallback, ale czcionka zaimportowana jako **Dynamic** renderuje
   każdy glif obecny w pliku. Zrób wspólne źródło czcionki (np. `UIFont` =>
   `lang ∈ {ru,ja,ko,zh} ? noto : latin`) i podepnij refresh pod zdarzenie
   zmiany języka, który przechodzi po wszystkich `Text` i przypisuje właściwą
   czcionkę. Etykiety samego przełącznika języka (日本語/한국어/中文/Русский)
   ustaw na czcionkę ICH pisma na stałe (inaczej w menu po angielsku = kwadraciki).

## Why this works
TMP rozwiązuje glify przez konfigurowalny łańcuch fallback (TextCore/FreeType),
więc jedna rejestracja fallbacku naprawia wszystkie brakujące pisma globalnie.
Legacy Text z czcionką Dynamic rasteryzuje glify z pliku TTF/OTF na żądanie —
wystarczy podać mu plik, który zawiera dany znak. Weryfikacja w edytorze bez
uruchamiania gry: `tmpFontAsset.TryAddCharacters("日本語한국어中文Русский", out
missing)` → `missing` puste = fallback fizycznie umie narysować te znaki.

## Trade-offs
- Rozmiar builda: regionalny Noto CJK ~16MB (do LFS: dodaj `*.otf filter=lfs`
  PRZED `git add`).
- Jeden plik CJK = domyślne kształty Han jednego regionu (japoński tekst dostanie
  lekko "chińskie" warianty wspólnych znaków). Akceptowalne dla indie; pełna
  poprawność = per-region fonty + przełączanie (więcej pracy).
- Dynamic TMP wymaga, by plik źródłowy był w buildzie (trzymaj go w `Resources/`
  — wtedy i legacy `Resources.Load<Font>` i TMP mają dostęp).

## Variants
- Trzy osobne Noto Sans JP/KR/SC jako fallbacki (mniejsze pojedynczo, ale
  unifikacja Han bierze pierwszy z listy → możliwe złe warianty; do cyrylicy i
  tak potrzebny dodatkowy font). Jeden Noto CJK jest prostszy.
- Static SDF zamiast Dynamic: trzeba z góry znać wszystkie znaki (przy CJK
  niepraktyczne — tysiące glifów). Dynamic jest właściwym wyborem dla CJK.
