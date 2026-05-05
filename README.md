# Whisper.cpp on Apple Silicon — Reproducible Benchmarks

Reproducible measurement series of [whisper.cpp](https://github.com/ggerganov/whisper.cpp) performance with the Metal GPU backend on Apple Silicon.

Published as part of the Mundwerk project (<https://mundwerkapp.de>) — a local dictation app that uses whisper.cpp in production. These measurements give developers, reviewers, and decision-makers an honest data foundation for setting realistic expectations about on-device Whisper.

## Status

✅ **Run 1 complete** — Apple M3 Ultra baseline (2026-05-05). See [`data/2026-05-05-r01/`](data/2026-05-05-r01/) for raw data and summary.

## Contents

- [`results-summary.md`](results-summary.md) — Aggregate table + headline findings
- [`methodology.md`](methodology.md) — Test setup, hardware specification, measurement parameters
- [`data/`](data/) — Raw data per run (`results.json`, `results.md`, `system.json`, `summary.md`)

## Headline Results — Apple M3 Ultra (11 s JFK sample, EN, 3 measurement runs)

| Model | Model size | Median inference | Real-time factor |
|-------|------------|------------------|------------------|
| `tiny` | 74 MB | 136 ms | **80.9× RT** |
| `base` | 141 MB | 162 ms | 68.1× RT |
| `small` | 465 MB | 310 ms | 35.5× RT |
| `medium` | 1463 MB | 714 ms | 15.4× RT |
| `large-v3-turbo` | 1549 MB | 623 ms | **17.7× RT** |

Notable: on M3 Ultra, `large-v3-turbo` is **faster than `medium`** despite the larger model file — the turbo variant uses a reduced decoder and is the sweet spot for interactive dictation.

📝 Long-form discussion: [Whisper.cpp on M3 Ultra — what it means for dictation](https://mundwerkapp.de/en/blog/whisper-benchmark-m3-ultra.html) (also [in German](https://mundwerkapp.de/blog/whisper-benchmark-m3-ultra.html))

## Planned Measurement Dimensions

1. **Model size:** tiny, base, small, medium, large-v3-turbo, large-v3
2. **Hardware:** M1, M1 Pro, M1 Max, M2, M2 Pro, M2 Max, M3, M3 Pro, M3 Max, M3 Ultra, M4 (at least one variant per family)
3. **Input audio:** 10 s, 30 s, 60 s, 5 min samples (German, English, mixed German-English)
4. **Backend modes:** Metal GPU (default), CPU-only (comparison baseline), CoreML (where applicable)
5. **Quantisation:** F16, Q8, Q5_K, Q4_K (where models are available)

## Measurement Targets

- **Latency** (wall-clock time from audio end to text result)
- **Throughput** (real-time factor: how many seconds of audio processed per second of wall-clock time)
- **WER** (Word Error Rate against human-verified transcript) — sampled per sample-set
- **Energy use** (via `powermetrics` analysis, joules per 60 s of audio)
- **Peak RAM footprint** (via `vm_stat` during inference)

## Contributing

Measurements on additional hardware configurations are welcome — please submit a Pull Request with:

1. Filled `data/<YYYY-MM-DD-rNN>/results.json` and `system.json` (schema in `data/2026-05-05-r01/`)
2. `summary.md` with headline findings
3. Reproducible invocation sequence documented

Conflict to the production app: Mundwerk itself is closed-source. These measurements only document the open-source inference layer (whisper.cpp), which is reproducible standalone.

## License

[Creative Commons Attribution 4.0](LICENSE) for data and text. Please cite as:

> Kindler, Bjoern (2026). *Whisper.cpp on Apple Silicon — Reproducible Benchmarks.*
> Run `<run-id>`. https://github.com/mundwerk-app/whisper-metal-benchmark

## Contact

**Bjoern Kindler** · <info@kindler-dev.de>
