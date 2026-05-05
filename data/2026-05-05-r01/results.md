# Mundwerk — Whisper.cpp Metal Benchmark

**Run:** 2026-05-05T08:12:52.423416+00:00  
**Host:** Apple M3 Ultra · macOS-26.4.1-arm64-arm-64bit-Mach-O  
**Sample:** `Libraries/whisper.cpp/samples/jfk.wav` (11.0 s, language=en)  
**Runs per model:** 3  

## Results

| Model | Size (MB) | Inference (ms, median) | Words | ms/word | RTF | Speed vs. real-time |
|-------|-----------|------------------------|-------|---------|-----|---------------------|
| tiny | 74.1 | 136.1 | 22 | 6.2 | 0.012 | 80.85× |
| base | 141.1 | 161.5 | 22 | 7.3 | 0.015 | 68.11× |
| small | 465.0 | 309.5 | 22 | 14.1 | 0.028 | 35.54× |
| medium | 1462.7 | 713.9 | 22 | 32.4 | 0.065 | 15.41× |
| large-v3-turbo | 1549.3 | 622.9 | 22 | 28.3 | 0.057 | 17.66× |

## Methodology

- whisper.cpp built with `-DGGML_METAL=ON -DGGML_ACCELERATE=ON -DCMAKE_BUILD_TYPE=Release`.
- Each model was run multiple times against a fixed audio sample. Median of inference time is reported.
- `RTF` = real-time factor (inference seconds / audio seconds). Lower is faster.
- `Speed vs. real-time` is the inverse: how many times faster than real-time the model runs.
- `ms/word` is median inference time divided by word count of the transcribed sample.

## Reproducibility

```
git rev-parse HEAD
# bc4ec24f0237ff9d06cdc30bccb21d50098876f6

python3 Scripts/whisper-bench.py --runs 3 --models tiny base small medium large-v3-turbo
```

_Sample:_ JFK clip, English, ~11 seconds, public-domain (whisper.cpp upstream).
