# Methodology

Test setup for reproducible whisper.cpp benchmarks on Apple Silicon.

## 1. Test Configuration

### 1.1 Software Stack

| Component | Version |
|-----------|---------|
| macOS | 26.x (current at measurement time, recorded per run) |
| whisper.cpp | tagged release (recorded per run in `data/<run>/system.json`) |
| Model source | <https://huggingface.co/ggerganov/whisper.cpp> |
| Build flags | `WHISPER_METAL=1`, `-O3`, ARM64 |

### 1.2 Hardware Profile per Run

Each run records a hardware profile as JSON:

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

Source: `system_profiler SPHardwareDataType` + `pmset -g thermlog`.

## 2. Audio Samples

Three curated sample sets, each in 10 s, 30 s, 60 s, and 5 min lengths:

- **DE-pure** — Standard German, clear pronunciation, neutral content (news anchor style)
- **EN-pure** — American English, neutral content
- **DE-EN-mixed** — German with English technical terms (code-switching) — Mundwerk's primary target case

Sample sources are stored as reproducible audio files in `data/samples/` (CC-BY licensed or self-produced).

All samples are normalised to 16 kHz, 16-bit PCM, mono (the standard Whisper input) — converted via `ffmpeg -ar 16000 -ac 1 -c:a pcm_s16le`.

## 3. Measurement Procedure

Per combination (hardware × model × sample × quantisation):

1. **Warm-up run:** 1 run discarded (model loads, caches warm up)
2. **5 measurement runs:** wall-clock time, peak RSS, GPU utilisation logged
3. **Reporting:** median + min/max of the 5 runs (no mean — robust against outliers)

Between runs: 30 s cooldown to avoid thermal-throttling effects. Before each sample set: check `pmset -g thermlog`; if `thermal_state != nominal`, insert a pause.

## 4. Measurement Tools

| Metric | Tool |
|--------|------|
| Wall-clock | `time` builtin (`real` value) |
| Peak RSS | `/usr/bin/time -l` (`maximum resident set size`) |
| GPU utilisation | `powermetrics --samplers gpu_power -i 100` |
| Energy | `powermetrics --samplers cpu_power,gpu_power -i 100` (joules via Δ × Δt) |
| Thermal | `pmset -g thermlog` |

## 5. WER Determination

Each sample has a human-verified reference transcript. WER is computed via [`jiwer`](https://github.com/jitsi/jiwer):

```python
from jiwer import wer
wer(reference_text, hypothesis_text)
```

Sample width: one WER value per model × sample length × sample language (not repeated per run, since Whisper with greedy decoding is deterministic).

## 6. Data Schema

CSV at `data/<run-id>/results.csv`:

```csv
run_id,timestamp_utc,chip,gpu_cores,memory_gb,model,quantization,sample_id,sample_length_s,sample_lang,backend,run_index,wall_clock_s,peak_rss_mb,gpu_util_avg_percent,energy_joule
2026-05-15-r01,2026-05-15T10:23:14Z,M2 Pro,19,32,large-v3,F16,de-pure-30s,30.0,de,metal,1,3.21,4923,72.3,18.4
...
```

## 7. Publication

After each run:

- Raw CSV in `data/<run-id>/`
- Hardware profile JSON in `data/<run-id>/system.json`
- Markdown summary in `data/<run-id>/summary.md`
- Aggregate table in `results-summary.md` (top entry = newest measurement)

Old runs are never overwritten — they also document performance drift across whisper.cpp versions.

## 8. Known Limitations

- **Not comparable to server GPUs.** These measurements document Apple Silicon only. NVIDIA A100 figures are not the goal; they are available in many other sources.
- **No audio-quality variations.** Tests run with clean studio audio. Real-world audio with background noise is not measured — it would distort WER incomparably and is a separate VAD topic anyway.
- **Sample size is small.** The goal is "understand the order of magnitude," not "statistical significance at the 95 % level." Anyone needing a scientific evaluation should use this material as a starting point for extended own measurements.

---

**As of:** 2026-05-05 · **Author:** Bjoern Kindler · **License:** [CC-BY-4.0](LICENSE)
