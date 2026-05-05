# Methodology

Versuchsaufbau für reproduzierbare Whisper.cpp-Benchmarks auf Apple Silicon.

## 1. Test-Konfiguration

### 1.1 Software-Stack

| Komponente | Version |
|------------|---------|
| macOS | 26.x (jeweils aktuell zum Messzeitpunkt, dokumentiert pro Lauf) |
| whisper.cpp | tagged release (siehe pro Lauf in `data/<run>/system.json`) |
| Modell-Quelle | <https://huggingface.co/ggerganov/whisper.cpp> |
| Build-Flags | `WHISPER_METAL=1`, `-O3`, ARM64 |

### 1.2 Hardware-Profil pro Lauf

Pro Lauf wird ein Hardware-Profil als JSON erfasst:

```json
{
  "chip": "Apple M2 Pro",
  "cores_performance": 8,
  "cores_efficiency": 4,
  "gpu_cores": 19,
  "memory_gb": 32,
  "macos_version": "26.4",
  "thermal_state": "nominal"
}
```

Quelle: `system_profiler SPHardwareDataType` + `pmset -g thermlog`.

## 2. Audio-Samples

Drei kuratierte Sample-Sets, jeweils in 10 s, 30 s, 60 s, 5 min Längen:

- **DE-pure** — Standarddeutsch, klare Aussprache, neutraler Inhalt (Nachrichtensprecher-Stil)
- **EN-pure** — American English, neutraler Inhalt
- **DE-EN-mixed** — Deutsch mit englischen Fachbegriffen (Code-Switching) — primärer Zielfall für Mundwerk

Sample-Quellen werden als reproduzierbare Audiofiles in `data/samples/` abgelegt (CC-BY-Lizenz oder Eigenproduktion).

Alle Samples werden in 16 kHz, 16-bit PCM, mono normiert (Standard-Whisper-Eingang) — Konvertierung via `ffmpeg -ar 16000 -ac 1 -c:a pcm_s16le`.

## 3. Mess-Prozedur

Pro Kombination (Hardware × Modell × Sample × Quantisierung):

1. **Warm-up-Run:** 1 Lauf wird verworfen (Modell wird geladen, Caches werden warm)
2. **5 Mess-Runs:** Wall-Clock-Zeit, Peak-RSS, GPU-Utilization protokolliert
3. **Reporting:** Median + Min/Max der 5 Runs (kein Mittelwert — robust gegen Ausreißer)

Zwischen Runs: 30 s Cooldown, um thermal-throttling-Effekte zu vermeiden. Vor jedem Sample-Set: `pmset -g thermlog` prüfen, bei `thermal_state != nominal` wird eine Pause eingelegt.

## 4. Mess-Werkzeuge

| Größe | Tool |
|-------|------|
| Wall-Clock | `time` builtin (`real` Wert) |
| Peak-RSS | `/usr/bin/time -l` (`maximum resident set size`) |
| GPU-Utilization | `powermetrics --samplers gpu_power -i 100` |
| Energy | `powermetrics --samplers cpu_power,gpu_power -i 100` (Joule via Δ × Δt) |
| Thermal | `pmset -g thermlog` |

## 5. WER-Bestimmung

Pro Sample existiert ein menschlich verifiziertes Referenztranskript. WER wird via [`jiwer`](https://github.com/jitsi/jiwer) berechnet:

```python
from jiwer import wer
wer(reference_text, hypothesis_text)
```

Stichprobenbreite: ein WER-Wert pro Modell × Sample-Length × Sample-Sprache (also nicht pro Run wiederholt, da Whisper mit Greedy-Decoding deterministisch ist).

## 6. Daten-Schema

CSV in `data/<run-id>/results.csv`:

```csv
run_id,timestamp_utc,chip,gpu_cores,memory_gb,model,quantization,sample_id,sample_length_s,sample_lang,backend,run_index,wall_clock_s,peak_rss_mb,gpu_util_avg_percent,energy_joule
2026-05-15-r01,2026-05-15T10:23:14Z,M2 Pro,19,32,large-v3,F16,de-pure-30s,30.0,de,metal,1,3.21,4923,72.3,18.4
...
```

## 7. Veröffentlichung

Nach jedem Lauf:

- Roh-CSV in `data/<run-id>/`
- Hardware-Profil-JSON in `data/<run-id>/system.json`
- Markdown-Zusammenfassung in `data/<run-id>/summary.md`
- Aggregat-Tabelle in `results-summary.md` (oberster Eintrag = neueste Messung)

Alte Läufe werden nie überschrieben — sie dokumentieren auch Performance-Drift über whisper.cpp-Versionen.

## 8. Bekannte Limitierungen

- **Nicht-vergleichbar mit Server-GPUs.** Diese Messungen dokumentieren ausschließlich Apple Silicon. NVIDIA-A100-Werte sind nicht das Ziel; sie sind in zahllosen anderen Quellen verfügbar.
- **Keine Audio-Quality-Variationen.** Tests laufen mit sauberem Studio-Audio. Real-World-Audio mit Hintergrundgeräuschen wird nicht gemessen — würde WER vergleichlos verzerren und ist ohnehin vorgelagertes VAD-Thema.
- **Stichprobengröße ist klein.** Ziel ist „Größenordnung verstehen", nicht „Statistische Signifikanz auf 95 %-Niveau". Wer eine wissenschaftliche Auswertung braucht, nehme dieses Material als Ausgangspunkt für eigene erweiterte Messungen.

---

**Stand:** 2026-05-05 · **Autor:** Bjoern Kindler · **Lizenz:** [CC-BY-4.0](LICENSE)
