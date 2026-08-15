---
type: lesson
project: Another Quest
suggested-category: pipeline/lessons
tags: [comfyui, vram, qwen-image, generacja-obrazkow, diagnostyka]
date: 2026-08-14
status: draft
severity: wysoka
---

# ComfyUI zawiesza się na 0/20 kroków, gdy model + enkoder nie mieszczą się w VRAM — leczy `--disable-smart-memory`

## Objaw

Sampler staje na `0%| 0/20 [00:00<?, ?it/s, Model Initializing ...]` i **nie rusza się przez 30 minut**.
Zero błędów, zero OOM, proces żyje, kolejka pokazuje „biegnie 1". Wygląda jak „wolno", jest jak „zakleszczone".

## Przyczyna

Qwen-Image fp8 = **19.5 GB**, enkoder tekstu Qwen2.5-VL fp8 = **7.9 GB**. Razem **27.4 GB na karcie 24 GB**.
Dynamic VRAM loading (domyślne od ComfyUI ~0.28) stara się trzymać oba naraz i wchodzi w thrashing
zamiast zwolnić enkoder po zakodowaniu promptu.

## Rozwiązanie

Start z `--disable-smart-memory` (wymusza zwalnianie modelu z VRAM, gdy nie jest używany).
Zmierzone na RTX 4090 24 GB: **30 min / 0 kroków → 57 s na obrazek 1024×1024, 20 kroków**.

    python main.py --listen 127.0.0.1 --port 8188 --disable-smart-memory --reserve-vram 1.0

`--lowvram` NIE pomaga — pomoc ComfyUI mówi wprost „doesn't do anything if dynamic vram is enabled".

## Dwa fałszywe tropy, które kosztowały czas

1. **„Inna aplikacja zżera VRAM"** — otwarty edytor Unity wyglądał na winowajcę (`memory.free` = 356 MiB).
   Pomiar PO ubiciu ComfyUI: 22.3 GB wolne przy nadal otwartym Unity, czyli edytor trzymał ~2 GB.
   **Mierz zużycie po wyłączeniu podejrzanego, nie przy włączonym** — inaczej przypiszesz winę
   pierwszemu procesowi, który widzisz.
2. **„Złe parametry workflow"** — podejrzenie padło na 1328×1328 i cfg 4.0 wzięte z blueprintu.
   Zejście na sprawdzone 1024×1024 / cfg 2.5 **nie pomogło**. Objaw był identyczny.

## Metoda, która rozstrzygnęła

Zamiast zgadywać parametry — **wyciągnięcie działającego workflow z metadanych wcześniejszych obrazków**.
ComfyUI zapisuje pełny graf w chunku `prompt` pliku PNG. Skan 120 ostatnich outputów dał odpowiedź na
pytanie „czy ten model kiedykolwiek tu działał" (115/120 użyć, ~80 s na obrazek) w jednym przebiegu.
To zamienia „czy to w ogóle działa na tej maszynie" z hipotezy w pomiar.

    # wyciagniecie grafu z PNG
    d = open(png,'rb').read(); i = d.find(b'prompt')
    s = d[i:i+60000]; s = s[s.find(b'{'):].split(b'\x00')[0].decode('utf-8','ignore')
    wf = json.loads(s[:s.rfind('}')+1])

## Reguła

Przy modelach, których suma (checkpoint + enkoder) przekracza VRAM karty, `--disable-smart-memory`
jest **domyślną flagą startową**, nie ratunkiem awaryjnym. Wpisać do skryptu startowego, nie do pamięci.

Powiązane: [[twierdzenie-z-jednej-probki-to-sygnal]], [[przeczytaj-i-napisz-u-nas-nie-wynajduj]]
