# Luce-Megakernel on RTX 4070, Plus the Bigger Story

This repository started from a Luce-Megakernel review, but the larger and more useful result from the last 48 hours is the comparative benchmark picture across TurboQuant, SpectralQuant, RotorQuant, GreenBoost, and RTX 4070 DVFS.

## Luce-Megakernel First, Briefly

`luce-org/luce-megakernel` is still the right conceptual entry point for this repo because it frames the original question well:

- persistent-kernel decode specialization is real and can matter
- architecture-specific hybrid decode paths can beat more general implementations
- batch-1 decode optimization is a meaningful design axis on consumer GPUs

On this host, though, the more important results were not "can Luce be copied directly?"

The more important results were:

1. how RotorQuant compares against TurboQuant and SpectralQuant on runnable 9B and 27B workloads
2. how far 122B can be pushed on an RTX 4070 once host-fallback and reduced envelopes are allowed
3. how much GreenBoost changes ranking by variant family
4. how much DVFS changes efficiency versus user-visible throughput

That is the focus of the rest of this document.

## Executive Summary

The strongest takeaways from the last 48 hours are:

1. Luce-style specialization remains interesting, but the immediate wins on this machine came from runtime-family behavior, not megakernel replication.
2. On Qwen3.5-9B, RotorQuant `iso3/f16` becomes the strongest overall row when GreenBoost is enabled.
3. On Qwen3.5-27B, RotorQuant is not a curiosity; it is directly competitive, and `iso3/f16` is the strongest no-GreenBoost row in this batch.
4. On Qwen3.5-122B, the full story splits into two envelopes:
   - reduced `ngl=10`, where RotorQuant `iso3/f16` is viable and strongest with GreenBoost on
   - patched `ngl=20`, where TurboQuant plus late-range host fallback makes the run possible, but SpectralQuant is near-parity rather than a raw throughput win
5. RTX 4070 DVFS matters, but modestly. `160 W` is the best practical default, `140 W` is the best pure efficiency point, and neither changes the system story as much as runtime choice does.

## Code Provenance, Forks, and Credits

This repository is a benchmark publication repo, not the main development repo.

That distinction matters because the results here were produced from multiple upstream and downstream codebases rather than from one single unified tree.

### Repositories used in this round

Primary code sources used directly in the last 48 hours:

| Role | Repository / tree | How it was used |
|---|---|---|
| Conceptual starting point | `luce-org/luce-megakernel` | Research reference for persistent-kernel decode specialization and hybrid-path thinking |
| Stock control runtime | `ggml-org/llama.cpp`-derived local stock tree | Baseline `llama-bench` comparison rows where runnable |
| TurboQuant / SpectralQuant runtime | local `llama-cpp-turboquant` tree | Main TQ and SQ benchmark/perplexity source, plus 122B host-fallback work |
| RotorQuant runtime | local `llama-cpp-rotorquant-llama` tree | Main RQ benchmark/perplexity source for `planar3` and `iso3` rows |
| GreenBoost shim/runtime environment | local `nvidia_greenboost` tree | Optional `LD_PRELOAD` runtime mode used for GreenBoost on/off comparisons |
| Publication repo | this repo | Final condensed write-up only |

Additional local port trees were inspected during this window, including `ports/llama-cpp-turboquant-b8756` and `ports/llama-cpp-rotorquant-b8756`, but those port builds were not clean enough to serve as benchmark sources for the published tables.

### What was actually merged

There were two different kinds of "merge" in this work, and they should not be confused.

1. Benchmark-level merge:
   - results from separate runtime families were normalized into matched benchmark envelopes and then compared in one report.
   - this is the main meaning of "merge" for the published tables.

2. Code-level selective carry-forward:
   - specific implementation ideas and fixes were carried into the active TurboQuant/SpectralQuant path where they were necessary to make new envelopes runnable.
   - the clearest example is the 122B `ngl=20` work, where allocator splitting and late-range host fallback were added in the TurboQuant/SQ branch so the model could run on this RTX 4070 host.

What did not happen:

- there was not a single monolithic source-tree merge of Luce, TurboQuant, SpectralQuant, RotorQuant, and stock llama.cpp into one benchmark binary.
- RotorQuant rows and TurboQuant/SQ rows still come from separate llama.cpp-family forks.
- Luce-Megakernel was not imported as benchmark code here; it was used as a design reference and framing influence.

### Practical method used to reconcile the forks

The method for this publication was:

1. Start from a Luce-style research question: would hybrid decode specialization matter on this machine?
2. Compare runnable local runtime families under matched envelopes instead of assuming the answer from architecture alone.
3. Keep stock, TQ/SQ, and RQ binaries separate when they were truly separate codebases.
4. Only carry forward code changes when they were needed to make a host envelope runnable or measurable.
5. Publish tables that clearly distinguish runtime families instead of pretending they came from one upstream.

That is why the report tables always label rows as Stock, TQ, SQ, or RQ rather than flattening everything into one synthetic benchmark family.

### Upstream and inspiration credits

Credit and thanks are due to the originators of the ideas and code that made this work possible.

| Project / group | Credit |
|---|---|
| `luce-org/luce-megakernel` | For the original prompt to look seriously at persistent decode specialization, hybrid-model kernels, and architecture-specific optimization instead of treating generic kernels as the only path |
| `ggml-org/llama.cpp` and contributors | For the base runtime, tooling, model-loading path, and benchmark utilities that all of these comparisons ultimately depend on |
| TurboQuant contributors | For the TQ runtime path and the practical K/V quantization variants used throughout the study |
| SpectralQuant contributors | For the SQ3 line of work, calibration tooling, and the compression path evaluated here |
| RotorQuant contributors | For the `planar3` and `iso3` runtime paths that produced some of the strongest results in this batch |
| GreenBoost work and inspiration | For the preload/runtime experimentation that made the GreenBoost on/off comparison axis possible |

### Thanks and acknowledgements

Thanks to the upstream authors and maintainers whose work provided both the baseline and the experimental paths here.

In particular:

- thanks to the Luce-Megakernel authors for the architectural inspiration;
- thanks to the `llama.cpp` maintainers and contributors for the underlying runtime foundation;
- thanks to the TurboQuant, SpectralQuant, and RotorQuant authors for exploring different K/V compression and runtime tradeoff spaces;
- and thanks to everyone doing the less glamorous systems work around loaders, allocators, offload boundaries, and runtime measurement, because on a 12 GB RTX 4070 that systems work turned out to matter as much as the compression schemes themselves.

## Scope and Metric Notes

This report covers benchmark and quality work completed in the last 48 hours, centered on:

- Qwen3.5-9B
- Qwen3.5-27B
- Qwen3.5-122B
- RTX 4070 DVFS on the 9B family

Metric notes:

- Prompt TPS and response TPS come from `llama-bench`.
- Peak VRAM for 27B and reduced 122B tables comes from the monitored perplexity runs, because those runs captured device-usage maxima cleanly.
- Peak VRAM for the 9B family comparison and the patched 122B `ngl=20` throughput matrix was not captured in the same structured form, so those cells are marked `n/c`.
- Wattage is only shown where it was actually measured. In practice that means the DVFS sweep.
- TTFT was not captured as a stable metric in these batches, so those cells are marked `n/c`.
- Perplexity is shown where it was actually measured. For the patched 122B `ngl=20` matrix, that cross-check was GreenBoost-on only.

Abbreviations used below:

- `GB`: GreenBoost
- `TQ`: TurboQuant family
- `SQ`: SpectralQuant
- `RQ`: RotorQuant
- `n/c`: not captured in this batch

## Model Metadata

| Model | GGUF file | Model class | File size | Params |
|---|---|---:|---:|---:|
| Qwen3.5-9B | `Qwen3.5-9B-Q4_K_M.gguf` | 9B | 5.23 GiB | 8.95B |
| Qwen3.5-27B | `Qwen3.5-27B-Q4_K_M.gguf` | 27B | 15.39 GiB | 26.90B |
| Qwen3.5-122B-A10B | `Qwen3.5-122B-A10B-IQ3_S-00001-of-00002.gguf` | 122B | 43.35 GiB | 122.11B |

## 1. Qwen3.5-9B Family Comparison

Envelope:

- `ngl=99`
- `p=512`
- `n=128`
- `r=2`
- `t=6`
- `b=512`
- `ub=512`
- `fa=0`

This is the first complete all-family comparison across stock, TurboQuant, SpectralQuant, and RotorQuant on this host.

| Model | Runtime | Variant | K/V | GB | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Qwen3.5-9B | Stock | baseline | `f16/f16` | off | 3154.7040 | 67.7588 | n/c | n/c | n/c | n/c | baseline stock row |
| Qwen3.5-9B | TQ | `tq1_k` | `q8_0/f16` | off | 3533.3924 | 70.3899 | n/c | n/c | n/c | n/c | strongest off-mode generation row |
| Qwen3.5-9B | TQ | `tq3_k` | `q4_0/f16` | off | 3229.4478 | 67.7758 | n/c | n/c | n/c | n/c | near-baseline generation |
| Qwen3.5-9B | SQ | `sq3_k` | `sq3/f16` | off | 3214.1993 | 66.9933 | n/c | n/c | n/c | n/c | operational, not speed leader |
| Qwen3.5-9B | RQ | `planar3_k` | `planar3/f16` | off | 3555.3492 | 67.4522 | n/c | n/c | n/c | n/c | strongest off-mode prompt row |
| Qwen3.5-9B | RQ | `iso3_k` | `iso3/f16` | off | 3245.1067 | 68.9004 | n/c | n/c | n/c | n/c | strongest balanced RQ off row |
| Qwen3.5-9B | Stock | baseline | `f16/f16` | on | 3190.3459 | 67.5688 | n/c | n/c | n/c | n/c | stock + GreenBoost |
| Qwen3.5-9B | TQ | `tq1_k` | `q8_0/f16` | on | 3249.9968 | 68.0006 | n/c | n/c | n/c | n/c | prompt edge shrinks under GB |
| Qwen3.5-9B | TQ | `tq3_k` | `q4_0/f16` | on | 3175.3591 | 68.1649 | n/c | n/c | n/c | n/c | slight generation gain vs stock |
| Qwen3.5-9B | SQ | `sq3_k` | `sq3/f16` | on | 3117.2023 | 68.2459 | n/c | n/c | n/c | n/c | close on gen, weak on prompt |
| Qwen3.5-9B | RQ | `planar3_k` | `planar3/f16` | on | 3223.8738 | 67.2031 | n/c | n/c | n/c | n/c | lost most off-mode prompt edge |
| Qwen3.5-9B | RQ | `iso3_k` | `iso3/f16` | on | 3298.7315 | 68.8726 | n/c | n/c | n/c | n/c | strongest overall 9B row with GB |

9B headline:

- Luce-style specialization is still an interesting future direction, but the practical result here was that RotorQuant `iso3/f16` became the best overall measured row with GreenBoost enabled.

## 2. Qwen3.5-27B Family Comparison

Envelope:

- `ngl=40`
- `p=64`
- `n=16`
- `r=2`
- `t=6`
- `b=64`
- `ub=64`
- `ctx-size=512`
- `fa=0`

This is one of the more interesting batches because it is large enough to be meaningful on a 4070, but still fully runnable across TurboQuant, SpectralQuant, and RotorQuant families.

| Model | Runtime | Variant | K/V | GB | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Qwen3.5-27B | Stock | baseline | `f16/f16` | off | 84.0966 | 4.0044 | 11347 | 3.5447 | n/c | n/c | stock baseline |
| Qwen3.5-27B | TQ | `tq1_k` | `q8_0/f16` | off | 86.4683 | 4.0416 | 11343 | 3.5432 | n/c | n/c | slightly better PPL than stock |
| Qwen3.5-27B | TQ | `tq3_k` | `q4_0/f16` | off | 86.5776 | 4.0095 | 11341 | 3.5416 | n/c | n/c | best measured PPL in batch |
| Qwen3.5-27B | SQ | `sq3_k` | `sq3/f16` | off | 88.3347 | 4.0107 | 11369 | 3.5526 | n/c | n/c | highest prompt in TQ/SQ family |
| Qwen3.5-27B | RQ | `planar3_k` | `planar3/f16` | off | 85.9979 | 4.2835 | 11347 | 3.5447 | n/c | n/c | best off-mode response TPS except iso3 |
| Qwen3.5-27B | RQ | `iso3_k` | `iso3/f16` | off | 96.5147 | 4.4015 | 11347 | 3.5447 | n/c | n/c | strongest overall off-mode row |
| Qwen3.5-27B | Stock | baseline | `f16/f16` | on | 81.5128 | 3.8989 | 11347 | 3.5447 | n/c | n/c | GB slightly hurts stock here |
| Qwen3.5-27B | TQ | `tq1_k` | `q8_0/f16` | on | 86.8784 | 3.9586 | 11343 | 3.5432 | n/c | n/c | best prompt inside TQ family |
| Qwen3.5-27B | TQ | `tq3_k` | `q4_0/f16` | on | 80.4237 | 4.0079 | 11341 | 3.5416 | n/c | n/c | best PPL, weaker prompt |
| Qwen3.5-27B | SQ | `sq3_k` | `sq3/f16` | on | 85.5268 | 3.8982 | 11369 | 3.5526 | n/c | n/c | prompt remains competitive, PPL small regression |
| Qwen3.5-27B | RQ | `planar3_k` | `planar3/f16` | on | 85.3749 | 4.0196 | 11347 | 3.5447 | n/c | n/c | stable under GB |
| Qwen3.5-27B | RQ | `iso3_k` | `iso3/f16` | on | 85.7083 | 4.0036 | 11347 | 3.5447 | n/c | n/c | still strong, but GB removes off-mode lead |

27B headline:

- RotorQuant is not just viable here; `iso3/f16` was the strongest no-GreenBoost row of the batch.
- SpectralQuant `sq3/f16` was competitive on prompt throughput, but not on quality-adjusted overall ranking.

## 3. Qwen3.5-122B Reduced Envelope (`ngl=10`)

Envelope:

- `ngl=10`
- `p=64`
- `n=16`
- `r=2`
- `t=6`
- `b=64`
- `ub=64`
- `ctx-size=512`
- `fa=0`

This is the strongest mixed-family 122B comparison that includes RotorQuant on this host.

| Model | Runtime | Variant | K/V | GB | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Qwen3.5-122B | TQ family | baseline | `f16/f16` | off | failed | failed | 10648 | 3.7456 | n/c | n/c | throughput load failure, perplexity run succeeded |
| Qwen3.5-122B | TQ family | `tq1_k` | `q8_0/f16` | off | 17.2433 | 1.3864 | 10475 | 3.7433 | n/c | n/c | weak throughput, best PPL in reduced set |
| Qwen3.5-122B | TQ family | `tq3_k` | `q4_0/f16` | off | 36.60 | 4.62 | 10473 | 3.7607 | n/c | n/c | strong throughput, worst PPL in reduced set |
| Qwen3.5-122B | SQ | `sq3_k` | `sq3/f16` | off | 14.5778 | 2.6665 | 10485 | 3.7543 | n/c | n/c | runnable, but slower than tq3 and iso3 |
| Qwen3.5-122B | RQ | `iso3_k` | `iso3/f16` | off | 34.46 | 4.62 | 10473 | 3.7456 | n/c | n/c | viable RotorQuant 122B path |
| Qwen3.5-122B | TQ family | baseline | `f16/f16` | on | 22.1654 | 3.1449 | 10475 | 3.7456 | n/c | n/c | baseline becomes runnable with GB |
| Qwen3.5-122B | TQ family | `tq1_k` | `q8_0/f16` | on | 23.1921 | 2.8967 | 10475 | 3.7433 | n/c | n/c | small PPL win, modest throughput |
| Qwen3.5-122B | TQ family | `tq3_k` | `q4_0/f16` | on | 26.5070 | 3.6300 | 10473 | 3.7607 | n/c | n/c | better prompt than baseline, weaker quality |
| Qwen3.5-122B | SQ | `sq3_k` | `sq3/f16` | on | 27.6100 | 1.7848 | 10485 | 3.7543 | n/c | n/c | prompt gain, large generation penalty |
| Qwen3.5-122B | RQ | `iso3_k` | `iso3/f16` | on | 35.02 | 4.47 | 10473 | 3.7456 | n/c | n/c | strongest measured reduced-envelope row |

122B reduced-envelope headline:

- RotorQuant `iso3/f16` is the viable RotorQuant 122B path on this RTX 4070.
- With GreenBoost on, it is the strongest measured reduced-envelope row overall.

## 4. Qwen3.5-122B Patched `ngl=20` Host-Fallback Matrix

Envelope:

- `ngl=20`
- `p=64`
- `n=16`
- `r=2`
- `t=6`
- `b=64`
- `ub=64`
- `fa=0`

This batch matters because it is the one that crossed the boundary from "hard failure" to "runnable" once late-range host fallback was wired in.

| Model | Runtime | Variant | K/V | GB | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Qwen3.5-122B | Stock | baseline | `f16/f16` | off | failed | failed | n/c | n/c | n/c | n/c | stock control failed |
| Qwen3.5-122B | Stock | baseline | `f16/f16` | on | failed | failed | n/c | n/c | n/c | n/c | stock control failed |
| Qwen3.5-122B | TQ family | baseline | `f16/f16` | off | 39.2716 | 5.2454 | n/c | n/c | n/c | n/c | patched host-fallback baseline |
| Qwen3.5-122B | TQ family | `tq1_k` | `q8_0/f16` | off | 35.1089 | 4.7314 | n/c | n/c | n/c | n/c | slower than patched baseline |
| Qwen3.5-122B | TQ family | `tq3_k` | `q4_0/f16` | off | 35.2601 | 4.7068 | n/c | n/c | n/c | n/c | slower than patched baseline |
| Qwen3.5-122B | SQ | `sq3_k` | `sq3/f16` | off | 34.2011 | 4.7362 | n/c | n/c | n/c | n/c | not a raw throughput win |
| Qwen3.5-122B | TQ family | baseline | `f16/f16` | on | 35.3835 | 4.6830 | n/c | 3.7456 | n/c | n/c | GreenBoost-on quality reference |
| Qwen3.5-122B | TQ family | `tq1_k` | `q8_0/f16` | on | 34.1332 | 4.6880 | n/c | 3.7445 | n/c | n/c | near-identical proxy PPL |
| Qwen3.5-122B | TQ family | `tq3_k` | `q4_0/f16` | on | 35.4716 | 4.3815 | n/c | 3.7607 | n/c | n/c | best GB-on prompt, worst GB-on gen |
| Qwen3.5-122B | SQ | `sq3_k` | `sq3/f16` | on | 35.1934 | 4.6760 | n/c | 3.7543 | n/c | n/c | near-throughput parity, near-baseline PPL |

122B patched-envelope headline:

- This is where the repo moves past the Luce framing and into systems work that mattered more on this host: allocator splitting plus late-range host fallback are what made `ngl=20` runnable.
- SpectralQuant `sq3/f16` did not win raw throughput here, but under GreenBoost it reached near-parity with the patched `f16/f16` baseline while staying close on proxy perplexity.

## 5. RTX 4070 DVFS Sweep

Envelope:

- `ngl=99`
- `p=512`
- `n=128`
- `r=3`
- `t=6`
- `b=512`
- `ub=512`
- `fa=0`

Measured cases:

- stock `f16/f16`
- RotorQuant `iso3/f16` with GreenBoost on

| Model | Runtime | Variant | K/V | Power Limit | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Gen tok/J |
|---|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Qwen3.5-9B | Stock | baseline | `f16/f16` | 220 | 3602.59 | 76.33 | n/c | n/c | 180.89 | n/c | 0.4220 |
| Qwen3.5-9B | Stock | baseline | `f16/f16` | 200 | 3614.89 | 76.37 | n/c | n/c | 191.02 | n/c | 0.3998 |
| Qwen3.5-9B | Stock | baseline | `f16/f16` | 180 | 3607.36 | 76.12 | n/c | n/c | 170.04 | n/c | 0.4477 |
| Qwen3.5-9B | Stock | baseline | `f16/f16` | 160 | 3529.45 | 75.73 | n/c | n/c | 148.94 | n/c | 0.5085 |
| Qwen3.5-9B | Stock | baseline | `f16/f16` | 140 | 3345.35 | 74.96 | n/c | n/c | 129.70 | n/c | 0.5779 |
| Qwen3.5-9B | RQ | `iso3_k` | `iso3/f16` | 220 | 3664.98 | 76.84 | n/c | n/c | 184.41 | n/c | 0.4167 |
| Qwen3.5-9B | RQ | `iso3_k` | `iso3/f16` | 200 | 3667.47 | 76.84 | n/c | n/c | 180.00 | n/c | 0.4269 |
| Qwen3.5-9B | RQ | `iso3_k` | `iso3/f16` | 180 | 3655.02 | 76.45 | n/c | n/c | 158.94 | n/c | 0.4810 |
| Qwen3.5-9B | RQ | `iso3_k` | `iso3/f16` | 160 | 3600.95 | 76.15 | n/c | n/c | 143.42 | n/c | 0.5308 |
| Qwen3.5-9B | RQ | `iso3_k` | `iso3/f16` | 140 | 3398.81 | 75.41 | n/c | n/c | 139.30 | n/c | 0.5414 |

DVFS headline:

- `160 W` is the best practical default on this GPU.
- `140 W` is the best pure efficiency point.
- The 4070 does show a real `tok/J` response, but not a dramatic runtime unlock.

## Negative and Boundary Findings That Matter

These are relevant because they explain what is not in the headline tables.

1. RotorQuant `planar3/f16` is not currently a viable 122B path on this host, including lower-offload probes.
2. Full-KV 122B rows (`q8_0/q8_0`, `q4_0/q4_0`) remain feasibility-envelope cases rather than stable headline rows on this RTX 4070.
3. The fresh 27B KV-dump rerun failed even with `LLAMA_ALLOW_KV_DUMP=1`, so the 27B report relies on throughput and perplexity artifacts rather than a fresh KV-dump calibration pass from this round.
4. Port-branch build attempts for the `b8756` TurboQuant and RotorQuant ports failed, so those port trees are not used as benchmark sources in this report.
5. Additional side smokes in this window, including the Mistral-Small-4-119B bench smoke and Gemma-3-12B KV-dump attempt, were not clean enough to include as primary result tables.

## Bottom Line

If this repository needs one concise conclusion, it should be this:

1. Luce-Megakernel remains a useful framing reference for decode-kernel specialization, but it was not the biggest story on this RTX 4070 host.
2. The bigger story was the runtime-family comparison:
   - RotorQuant `iso3/f16` won the 9B GreenBoost-on comparison
   - RotorQuant was directly competitive on 27B
   - RotorQuant `iso3/f16` is the viable 122B reduced-envelope path
   - SpectralQuant `sq3/f16` on patched 122B `ngl=20` reached near-parity rather than a decisive speed win
3. GreenBoost changes family ranking more than it changes the existence of the ranking.
4. DVFS helps efficiency, but the runtime-family choice still matters more than power-limit tuning on this specific card.

## Source Runs

Primary run roots used for this report:

- `spectralquant_study/runs/20260409T155221Z_qwen9b-k-only-all-families-off`
- `spectralquant_study/runs/20260409T155508Z_qwen9b-k-only-all-families-on`
- `spectralquant_study/runs/20260409T140526Z_qwen122b_ngl20_quant_greenboost_matrix`
- `spectralquant_study/runs/20260409T195937Z_qwen122b-ngl10-ppl-vram-reduced`
- `spectralquant_study/runs/20260409T161412Z_qwen122b-ngl10-iso3-reduced-off`
- `spectralquant_study/runs/20260409T161418Z_qwen122b-ngl10-iso3-reduced-on`
- `spectralquant_study/runs/20260409T210520Z_qwen27b-k-only-all-families-off`
- `spectralquant_study/runs/20260409T210520Z_qwen27b-k-only-all-families-on`
- `spectralquant_study/runs/20260409T211617Z_qwen27b-ppl-vram-all-families`
