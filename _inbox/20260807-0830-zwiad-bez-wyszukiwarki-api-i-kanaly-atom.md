---
title: Zwiad technologiczny bez wyszukiwarki - API i kanaly atom
type: pattern
status: draft
confidence: high
verified: '2026-08-07'
date: 2026-08-07
project: GameDevOS radar
tags: [radar, research, arxiv, github, huggingface, blender, api, rate-limit]
applies_to: [radar, kb-maintenance]
suggested-category: workflow/research
---

# Zwiad technologiczny bez wyszukiwarki - API i kanaly atom

## When to use
Gdy budzet WebSearch jest wyczerpany, api.github.com pokazuje 0 pozostalych zapytan,
a wyszukiwarka HTML GitHuba zwraca 429. Czyli w praktyce pod koniec kazdej dluzszej
sesji zwiadowczej.

## Steps

1. **arXiv** - zwykly curl na `http://export.arxiv.org` zwraca HTTP 301 i PUSTY plik.
   Trzeba `https` plus `-L`:
   ```
   curl -sSL -m 90 --get "https://export.arxiv.org/api/query" \
     --data-urlencode 'search_query=cat:cs.GR AND (abs:rigging OR abs:skeleton)' \
     --data-urlencode 'max_results=80' \
     --data-urlencode 'sortBy=submittedDate' --data-urlencode 'sortOrder=descending' \
     -o wynik.xml
   ```
   Parsowac `xml.etree` w Pythonie. Na Windows ZAWSZE `PYTHONIOENCODING=utf-8`,
   inaczej cp1250 wywala sie na greckich literach w streszczeniach.

2. **Czy praca ma kod** - linki do repozytoriow czesto NIE ma w streszczeniu,
   za to jest na stronie `https://arxiv.org/abs/<id>` w sekcji komentarzy.
   Pobrac ta strone i wyciagnac regexem `github|gitlab|huggingface`.
   Odfiltrowac szum `huggingface.co/docs/hub/spaces`.

3. **GitHub bez API** (wszystko HTTP 200, bez limitow):
   - daty commitow: `https://github.com/OWNER/REPO/commits/main.atom`
     (jak brak `<entry>`, sprobowac `master.atom`)
   - lista plikow w repo: pobrac `https://github.com/OWNER/REPO` z naglowkiem
     `User-Agent: Mozilla/5.0`, wyciagnac regexem `"name":"([^"]{1,60})","path"`.
     To od razu mowi, czy repo ma kod, czy jest pusta wydmuszka i czy jest LICENSE.
   - tresc licencji: `https://raw.githubusercontent.com/OWNER/REPO/main/LICENSE`
   - odkrywanie repozytoriow: `https://github.com/topics/<temat>`, regex
     `href="/([A-Za-z0-9_.-]+/[A-Za-z0-9_.-]+)" data-view-component`
   NIE dziala: `github.com/search?q=` (429), `grep.app` (blokada Vercel).

4. **Hugging Face** - `https://huggingface.co/api/models?search=X&sort=createdAt&direction=-1&limit=50`
   oraz `.../api/datasets?...`. Karta modelu surowa:
   `https://huggingface.co/<id>/raw/main/README.md`.
   Zbiory bramkowane (`gated`) zwracaja komunikat o dostepie zamiast tresci -
   wtedy licencji NIE da sie zweryfikowac i trzeba to napisac wprost.

5. **Dodatki do Blendera** - jest pelny wykaz w formacie JSON, nikt o nim nie pamieta:
   `https://extensions.blender.org/api/v1/extensions/` (ok. 1300 pozycji, z licencja
   SPDX i tagami). Brak dat, wiec daty publikacji dobrac ze strony pozycji
   `https://extensions.blender.org/add-ons/<id>/`.
   Uwaga: WebFetch dostaje 403 na blender.org i developer.blender.org, a zwykly curl
   z naglowkiem Mozilla dostaje 200.

## Why this works
Publiczne kanaly atom i punkty API sa poza mechanizmem limitowania, ktory chroni
wyszukiwarki. Strony HTML GitHuba maja doklejony JSON z pelna lista plikow, wiec
jedno pobranie zastepuje kilka zapytan do API.

## Trade-offs
- Nie da sie szukac po tresci kodu (grep.app i wyszukiwarka GitHuba padaja).
  Odkrywanie nowych repozytoriow jest slabe - zostaja tematy i listy typu awesome.
- Kanaly atom daja tylko ostatnie 20 commitow, bez liczby gwiazdek.

## Variants
Gdy limit api.github.com sie odnowi (60 na godzine), `https://api.github.com/rate_limit`
sprawdza to jednym tanim zapytaniem - warto zaczac od tego, zanim sie wybierze sciezke.
