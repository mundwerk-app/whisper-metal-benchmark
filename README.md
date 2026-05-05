# Whisper.cpp on Apple Silicon — Reproducible Benchmarks

Reproduzierbare Messreihen zur Performance von [whisper.cpp](https://github.com/ggerganov/whisper.cpp) mit Metal-GPU-Backend auf Apple Silicon.

Veröffentlicht im Rahmen des Mundwerk-Projekts (<https://mundwerkapp.de>) — eine lokale Diktiersoftware, die whisper.cpp produktiv einsetzt. Diese Messungen sollen Entwicklern, Reviewern und Entscheidern eine ehrliche Datengrundlage geben, um Erwartungen an On-Device-Whisper realistisch einzuschätzen.

## Status

🚧 **In Vorbereitung** — Skelett, Methodik und Messplan stehen. Erste vollständige Datenerhebung folgt mit der nächsten App-Release-Welle.

## Inhalt

- [`methodology.md`](methodology.md) — Versuchsaufbau, Hardware-Spezifikation, Messparameter
- [`data/`](data/) — CSV-Rohdaten der Messreihen (kommen ab Lauf 1)
- [`results-summary.md`](results-summary.md) — Zusammenfassende Tabellen und Plots *(folgt)*

## Quick-Reference (Vor-Ergebnis aus Mundwerk-Praxis, M2-Klasse)

> Diese Tabelle ist ein **Erwartungswert aus dem App-Einsatz**, keine kontrollierte Messung. Sie wird durch die Benchmark-Daten in `data/` ersetzt, sobald Lauf 1 abgeschlossen ist.

| Modell | Modellgröße auf Disk | RAM-Footprint (geschätzt) | Latenz für 10 s Audio (Realtime-Faktor) |
|--------|----------------------|---------------------------|------------------------------------------|
| `tiny`     | ~75 MB | ~0,5 GB | <0,5 s (~20× RT) |
| `base`     | ~150 MB | ~1 GB | ~1 s (~10× RT) |
| `small`    | ~500 MB | ~1,5 GB | ~2 s (~5× RT) |
| `medium`   | ~1,5 GB | ~3 GB | ~3 s (~3,3× RT) |
| `large-v3` | ~3 GB | ~5 GB | ~4 s (~2,5× RT) |

## Geplante Messdimensionen

1. **Modellgröße:** tiny, base, small, medium, large-v3
2. **Hardware:** M1, M1 Pro, M1 Max, M2, M2 Pro, M2 Max, M3, M3 Pro, M3 Max, M4 (mindestens je eine Variante)
3. **Eingangs-Audio:** 10 s, 30 s, 60 s, 5 min Samples (deutsch, englisch, deutsch-englisch gemischt)
4. **Backend-Modi:** Metal-GPU (Standard), CPU-only (Vergleichsbasis), CoreML (falls verfügbar)
5. **Quantisierung:** F16, Q8, Q5_K, Q4_K (sofern Modell verfügbar)

## Messziele

- **Latenz** (Wall-Clock-Zeit von Audio-Ende bis Text-Ergebnis)
- **Throughput** (Realtime-Faktor: wie viele Audio-Sekunden pro Sekunde Wall-Clock verarbeitet werden)
- **WER** (Word Error Rate gegen menschlich verifiziertes Transkript) — Stichprobe pro Sample-Set
- **Energy Use** (über `powermetrics`-Auswertung, Joule pro 60 s Audio)
- **Peak-RAM-Footprint** (über `vm_stat` während Inference)

## Mitwirken

Eigene Messungen auf weiteren Hardware-Konfigurationen sind willkommen — bitte als Pull Request mit:

1. Befüllter Data-CSV nach Schema in `methodology.md` §6
2. System-Profil (Output von `system_profiler SPHardwareDataType`, anonymisiert)
3. Messpipeline-Aufruf-Log

Konflikt zur produktiv-App: Mundwerk selbst ist closed-source. Diese Messungen dokumentieren ausschließlich den Open-Source-Inference-Layer (whisper.cpp), der auch ohne Mundwerk reproduzierbar ist.

## Lizenz

[Creative Commons Attribution 4.0](LICENSE) für Daten und Texte. Bitte zitieren als:

> Kindler, Bjoern (2026). *Whisper.cpp on Apple Silicon — Reproducible Benchmarks.*
> https://github.com/mundwerk-app/whisper-metal-benchmark

## Kontakt

**Bjoern Kindler** · <info@kindler-dev.de>
