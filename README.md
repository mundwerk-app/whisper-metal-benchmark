# Whisper.cpp on Apple Silicon — Reproducible Benchmarks

Reproduzierbare Messreihen zur Performance von [whisper.cpp](https://github.com/ggerganov/whisper.cpp) mit Metal-GPU-Backend auf Apple Silicon.

Veröffentlicht im Rahmen des Mundwerk-Projekts (<https://mundwerkapp.de>) — eine lokale Diktiersoftware, die whisper.cpp produktiv einsetzt. Diese Messungen sollen Entwicklern, Reviewern und Entscheidern eine ehrliche Datengrundlage geben, um Erwartungen an On-Device-Whisper realistisch einzuschätzen.

## Status

✅ **Lauf 1 abgeschlossen** — Apple M3 Ultra Baseline (2026-05-05). Siehe [`data/2026-05-05-r01/`](data/2026-05-05-r01/) für Rohdaten + Summary.

## Inhalt

- [`results-summary.md`](results-summary.md) — Aggregat-Tabelle + Headline-Findings
- [`methodology.md`](methodology.md) — Versuchsaufbau, Hardware-Spezifikation, Messparameter
- [`data/`](data/) — Rohdaten je Lauf (`results.json`, `results.md`, `system.json`, `summary.md`)

## Headline-Resultate Apple M3 Ultra (11 s JFK-Sample, EN, 3 Mess-Läufe)

| Modell | Modellgröße | Median Inference | Realtime-Faktor |
|--------|-------------|------------------|-----------------|
| `tiny` | 74 MB | 136 ms | **80.9× RT** |
| `base` | 141 MB | 162 ms | 68.1× RT |
| `small` | 465 MB | 310 ms | 35.5× RT |
| `medium` | 1463 MB | 714 ms | 15.4× RT |
| `large-v3-turbo` | 1549 MB | 623 ms | **17.7× RT** |

Auffällig: `large-v3-turbo` ist auf M3 Ultra **schneller als `medium`** trotz größerer Modelldatei — Turbo-Variante nutzt reduzierten Decoder und ist für interaktives Diktat der Sweet-Spot.

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
