# Results Summary

Aggregierte Ergebnisse aller Mess-Läufe. Neueste oben.

## Lauf-Index

| Run-ID | Datum | Hardware | Whisper.cpp | Modelle | Status |
|--------|-------|----------|-------------|---------|--------|
| [2026-05-05-r01](data/2026-05-05-r01/) | 2026-05-05 | Apple M3 Ultra (80 GPU cores, 512 GB) | v1.8.4 | tiny · base · small · medium · large-v3-turbo | ✅ Abgeschlossen |

## Apple Silicon Übersicht (Realtime-Faktor je Modell)

> Realtime-Faktor = wie viele Sekunden Audio pro Sekunde Wall-Clock verarbeitet werden. Höher = schneller. Median über 3 Läufe.

| Hardware | tiny | base | small | medium | large-v3-turbo |
|----------|------|------|-------|--------|----------------|
| M3 Ultra (80 GPU) | 80.9× | 68.1× | 35.5× | 15.4× | 17.7× |
| *M1 / M2 / M3 base / Pro / Max* | *PR welcome* | | | | |

## Headline-Erkenntnisse (Stand 2026-05-05)

1. **`large-v3-turbo` schlägt `medium`** auf M3 Ultra: 17.7× vs 15.4× Realtime, bei vergleichbarer Modellgröße. Für produktives Diktat = der Sweet-Spot.
2. **Selbst `tiny` läuft 80× real-time** — auf der Top-End-M3-Ultra-Hardware ist Inference niemals der Bottleneck, nur Model-Load (einmalig, <2 s für medium-tier).
3. **Skalierung** der Latenz in ms/Wort ist nahezu linear in Modellgröße bis `medium`; `large-v3-turbo` durchbricht das Muster durch reduzierten Decoder.

## Methodik

Siehe [`methodology.md`](methodology.md). Wichtigste Punkte:

- whisper.cpp `-DGGML_METAL=ON -DGGML_ACCELERATE=ON -DCMAKE_BUILD_TYPE=Release`
- 3 Mess-Läufe pro Modell × Sample, Median berichtet (kein Mittelwert — robust gegen Ausreißer)
- JFK-Sample (11 s, Englisch, 22 Worte) als Referenz

## Mitwirken

Eigene Messungen auf weiteren Apple-Silicon-Konfigurationen sind willkommen — bitte als Pull Request mit:

1. Befüllter `data/<YYYY-MM-DD-rNN>/results.json` + `system.json` (Schema siehe `data/2026-05-05-r01/`)
2. `summary.md` mit Headline-Findings
3. Reproduzierbare Aufruf-Sequenz dokumentiert

Besonders gesucht: M1, M1 Pro/Max, M2, M2 Pro/Max, M3, M3 Pro/Max, M4 Familie.

## Zitieren

> Kindler, Bjoern (2026). *Whisper.cpp on Apple Silicon — Reproducible Benchmarks.*
> Run `<run-id>`.
> https://github.com/mundwerk-app/whisper-metal-benchmark

---

**Stand:** 2026-05-05 · **Maintainer:** Bjoern Kindler · **Lizenz:** [CC-BY-4.0](LICENSE)
