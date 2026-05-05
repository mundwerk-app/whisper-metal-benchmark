# Results Summary

Aggregated results across all measurement runs. Newest first.

## Run Index

| Run ID | Date | Hardware | whisper.cpp | Models | Status |
|--------|------|----------|-------------|--------|--------|
| [2026-05-05-r01](data/2026-05-05-r01/) | 2026-05-05 | Apple M3 Ultra (80 GPU cores, 512 GB) | v1.8.4 | tiny · base · small · medium · large-v3-turbo | ✅ Complete |

## Apple Silicon Overview (Real-Time Factor per Model)

> Real-time factor = how many seconds of audio are processed per second of wall-clock time. Higher = faster. Median over 3 runs.

| Hardware | tiny | base | small | medium | large-v3-turbo |
|----------|------|------|-------|--------|----------------|
| M3 Ultra (80 GPU) | 80.9× | 68.1× | 35.5× | 15.4× | 17.7× |
| *M1 / M2 / M3 base / Pro / Max* | *PR welcome* | | | | |

## Headline Findings (as of 2026-05-05)

1. **`large-v3-turbo` beats `medium`** on M3 Ultra: 17.7× vs 15.4× real-time, at comparable model size on disk. For productive dictation = the sweet spot.
2. **Even `tiny` runs at 80× real-time** — on top-end M3 Ultra hardware, inference is never the bottleneck, only model load (one-time, <2 s for medium tier).
3. **Latency scaling** in ms/word is nearly linear in model size up to `medium`; `large-v3-turbo` breaks the pattern via reduced decoder.

## Methodology

See [`methodology.md`](methodology.md). Key points:

- whisper.cpp built with `-DGGML_METAL=ON -DGGML_ACCELERATE=ON -DCMAKE_BUILD_TYPE=Release`
- 3 measurement runs per model × sample, median reported (no mean — robust against outliers)
- JFK sample (11 s, English, 22 words) as reference

## Contributing

Measurements on additional Apple Silicon configurations are welcome — please submit a Pull Request with:

1. Filled `data/<YYYY-MM-DD-rNN>/results.json` + `system.json` (schema in `data/2026-05-05-r01/`)
2. `summary.md` with headline findings
3. Documented reproducible invocation sequence

Particularly wanted: M1, M1 Pro/Max, M2, M2 Pro/Max, M3, M3 Pro/Max, M4 family.

## Citation

> Kindler, Bjoern (2026). *Whisper.cpp on Apple Silicon — Reproducible Benchmarks.*
> Run `<run-id>`.
> https://github.com/mundwerk-app/whisper-metal-benchmark

---

**As of:** 2026-05-05 · **Maintainer:** Bjoern Kindler · **License:** [CC-BY-4.0](LICENSE)
