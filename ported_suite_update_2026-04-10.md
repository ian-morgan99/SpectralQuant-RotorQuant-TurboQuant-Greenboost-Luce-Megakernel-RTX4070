# April 10 Ported Benchmark Suite Update

This note publishes the April 10 rerun of the ported benchmark suite.

It is intentionally separate from the main [README.md](README.md): the README remains the broader narrative and cross-model summary, while this file records the same-day rerun of the ported trees and the exact outcomes of that batch.

## What Was Rerun

The April 10 driver reran the k-only all-family benchmark matrix against ported trees and matched manifests for six model families:

1. Qwen3.5-9B
2. Qwen3.5-27B
3. Qwen3.5-122B-A10B
4. Gemma 4 31B
5. NVIDIA Nemotron 120B-A12B
6. Mistral Small 4 119B

The driver script used was:

- `spectralquant_study/scripts/run_ported_benchmark_suite_20260410.sh`

Important scope note:

- the first publication pass for this note covered the GreenBoost-off throughput rerun only;
- later April 10 runs filled in GreenBoost-on throughput, most of the fresh ctx=512 perplexity / peak-VRAM matrix, and several 2048-context checks;
- the sections below keep the original off-only rerun intact and then append the later same-day gapfill results;
- remaining gaps are called out explicitly where they still exist.

## Driver Outcome Summary

The first-pass driver summary was:

| Label | Initial status | Notes |
|---|---|---|
| `qwen9b-k-only-all-families-ported-off` | ok | first pass usable |
| `qwen27b-k-only-all-families-ported-off` | failed | rerun produced usable result set |
| `qwen122b-k-only-all-families-ported-off` | ok | first pass usable |
| `gemma4-31b-k-only-all-families-ported-off` | ok | first pass usable |
| `nemotron120b-k-only-all-families-ported-off` | ok | first pass usable |
| `mistral119b-k-only-all-families-ported-off` | failed | later reruns converged to final usable result set |

The rerun / final usable run roots referenced in this note are:

- Qwen3.5-9B: `spectralquant_study/runs/20260410T072755Z_qwen9b-k-only-all-families-ported-off`
- Qwen3.5-27B: `spectralquant_study/runs/20260410T093144Z_qwen27b-k-only-all-families-ported-off-rerun`
- Qwen3.5-122B-A10B: `spectralquant_study/runs/20260410T073749Z_qwen122b-k-only-all-families-ported-off`
- Gemma 4 31B: `spectralquant_study/runs/20260410T074043Z_gemma4-31b-k-only-all-families-ported-off`
- Nemotron 120B-A12B: `spectralquant_study/runs/20260410T081405Z_nemotron120b-k-only-all-families-ported-off`
- Mistral Small 4 119B: `spectralquant_study/runs/20260410T115510Z_mistral119b-k-only-all-families-ported-off-final`

## Method

The suite used the standard matrix runner with GreenBoost disabled and model-specific manifests.

Representative driver behavior:

- SQ3 basis binaries were generated for the models that needed them.
- Stock, TurboQuant, and RotorQuant benchmark binaries were passed to the matrix runner.
- Results were recorded as raw `llama_bench/*.jsonl` outputs per variant.

Two envelope classes were used:

### Large prompt/generation envelope

Used for:

- Qwen3.5-9B
- Qwen3.5-27B
- Gemma 4 31B

Representative settings:

- `p=512`
- `n=128`
- `flash_attn=1`
- `batch=512`
- `ubatch=128`

### Conservative large-model smoke / throughput envelope

Used for:

- Qwen3.5-122B-A10B
- Nemotron 120B-A12B
- Mistral Small 4 119B

Representative settings:

- `p=1`
- `n=1`
- `flash_attn=0`
- `batch=64`
- `ubatch=64`
- `ngl=10`

## Result Tables

Legend:

- Runtime family labels:
  - Stock: baseline `f16/f16`
  - TQ: `q8_0/f16`, `q4_0/f16`
  - SQ: `sq3/f16`
  - RQ: `planar3/f16`, `iso3/f16`
- `n/c` means not captured in this April 10 rerun batch.

## Qwen3.5-9B Ported Rerun

Metadata:

- model type: `qwen35 9B Q4_K - Medium`
- model size: `5.23 GiB`
- params: `8.95B`
- offload depth: `ngl=999`

| Model | Runtime | Variant | K/V | GB | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Qwen3.5-9B | Stock | baseline | `f16/f16` | off | 2936.6267 | 76.0344 | n/c | n/c | n/c | n/c | strongest baseline response row |
| Qwen3.5-9B | TQ | `tq1_k` | `q8_0/f16` | off | 1372.9995 | 68.6574 | n/c | n/c | n/c | n/c | much weaker prompt than baseline in this ported rerun |
| Qwen3.5-9B | TQ | `tq3_k` | `q4_0/f16` | off | 1334.1907 | 66.4767 | n/c | n/c | n/c | n/c | slowest response row in this batch |
| Qwen3.5-9B | SQ | `sq3_k` | `sq3/f16` | off | 1340.2632 | 69.4655 | n/c | n/c | n/c | n/c | SQ remained well below baseline prompt throughput |
| Qwen3.5-9B | RQ | `planar3_k` | `planar3/f16` | off | 3110.6041 | 71.9445 | n/c | n/c | n/c | n/c | strongest prompt row in this ported rerun |
| Qwen3.5-9B | RQ | `iso3_k` | `iso3/f16` | off | 2976.7540 | 73.0572 | n/c | n/c | n/c | n/c | strongest balanced non-baseline row |

Review note:

- In this ported-off rerun, RotorQuant remained the standout family for prompt throughput.
- TQ and SQ prompt throughput were much lower than the earlier non-ported 9B all-family result set, so this rerun should be read as a ported-tree validation result, not a replacement for the prior broader benchmark story.

## Qwen3.5-27B Ported Rerun

Metadata:

- model type: `qwen35 27B Q4_K - Medium`
- model size: `15.39 GiB`
- params: `26.90B`
- offload depth: `ngl=10`

| Model | Runtime | Variant | K/V | GB | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Qwen3.5-27B | Stock | baseline | `f16/f16` | off | 93.0249 | 2.2085 | n/c | n/c | n/c | n/c | best prompt row in rerun |
| Qwen3.5-27B | TQ | `tq1_k` | `q8_0/f16` | off | 83.5727 | 2.2021 | n/c | n/c | n/c | n/c | below baseline on both axes |
| Qwen3.5-27B | TQ | `tq3_k` | `q4_0/f16` | off | 84.4066 | 2.2029 | n/c | n/c | n/c | n/c | slightly ahead of tq1 on prompt |
| Qwen3.5-27B | SQ | `sq3_k` | `sq3/f16` | off | 82.9574 | 2.2048 | n/c | n/c | n/c | n/c | weakest prompt row in rerun |
| Qwen3.5-27B | RQ | `planar3_k` | `planar3/f16` | off | 89.2797 | 2.2104 | n/c | n/c | n/c | n/c | best non-baseline response tie region |
| Qwen3.5-27B | RQ | `iso3_k` | `iso3/f16` | off | 91.2289 | 2.2155 | n/c | n/c | n/c | n/c | strongest non-baseline row overall |

Review note:

- The rerun again favored RotorQuant over TQ and SQ on this model.
- Unlike the earlier 27B batch, the baseline row stayed strongest on prompt throughput in the ported rerun, while `iso3/f16` remained the best non-baseline row.

## Qwen3.5-122B-A10B Ported Rerun

Metadata:

- model type: `qwen35moe 122B.A10B Q6_K`
- model size: `43.35 GiB`
- params: `122.11B`
- offload depth: `ngl=10`

| Model | Runtime | Variant | K/V | GB | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Qwen3.5-122B | Stock | baseline | `f16/f16` | off | 4.4475 | 4.5471 | n/c | n/c | n/c | n/c | baseline usable in this rerun |
| Qwen3.5-122B | TQ | `tq1_k` | `q8_0/f16` | off | 4.4266 | 4.4791 | n/c | n/c | n/c | n/c | near-baseline |
| Qwen3.5-122B | TQ | `tq3_k` | `q4_0/f16` | off | 4.5490 | 4.6171 | n/c | n/c | n/c | n/c | strongest TQ row |
| Qwen3.5-122B | SQ | `sq3_k` | `sq3/f16` | off | 4.5029 | 4.5361 | n/c | n/c | n/c | n/c | close to baseline on both axes |
| Qwen3.5-122B | RQ | `planar3_k` | `planar3/f16` | off | 4.2405 | 4.7415 | n/c | n/c | n/c | n/c | best response throughput, weaker prompt |
| Qwen3.5-122B | RQ | `iso3_k` | `iso3/f16` | off | 3.6150 | 4.1484 | n/c | n/c | n/c | n/c | weakest row in this rerun |

Review note:

- This ported rerun differs from the earlier reduced-envelope story where `iso3/f16` was the strongest GreenBoost-on row.
- In the April 10 off-only ported run, `tq3_k` and `planar3_k` looked stronger than `iso3_k`.
- That makes this an important cross-check rather than a trivial confirmation run.

## Gemma 4 31B Ported Rerun

Metadata:

- model type: `gemma4 ?B Q8_0`
- model size: `30.38 GiB`
- params: `30.70B`
- offload depth: `ngl=10`

| Model | Runtime | Variant | K/V | GB | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Gemma 4 31B | Stock | baseline | `f16/f16` | off | 46.3066 | 1.1072 | n/c | n/c | n/c | n/c | baseline slowest gen row |
| Gemma 4 31B | TQ | `tq1_k` | `q8_0/f16` | off | 46.0407 | 1.2338 | n/c | n/c | n/c | n/c | response gain without prompt gain |
| Gemma 4 31B | TQ | `tq3_k` | `q4_0/f16` | off | 46.8038 | 1.2335 | n/c | n/c | n/c | n/c | modest prompt uplift |
| Gemma 4 31B | SQ | `sq3_k` | `sq3/f16` | off | 46.8115 | 1.2296 | n/c | n/c | n/c | n/c | near tq3 on prompt |
| Gemma 4 31B | RQ | `planar3_k` | `planar3/f16` | off | 49.5155 | 1.2349 | n/c | n/c | n/c | n/c | strong prompt uplift |
| Gemma 4 31B | RQ | `iso3_k` | `iso3/f16` | off | 50.7926 | 1.2315 | n/c | n/c | n/c | n/c | strongest prompt row overall |

Review note:

- RotorQuant again dominated prompt throughput on the medium-large model in the off-only rerun.
- Response throughput was tightly clustered among all compressed variants and above stock.

## Nemotron 120B-A12B Ported Rerun

Metadata:

- model type: `nemotron_h_moe 120B.A12B Q2_K - Medium`
- model size: `50.89 GiB`
- params: `120.67B`
- offload depth: `ngl=10`

| Model | Runtime | Variant | K/V | GB | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Nemotron 120B | Stock | baseline | `f16/f16` | off | 3.1762 | 3.1321 | n/c | n/c | n/c | n/c | stock baseline |
| Nemotron 120B | TQ | `tq1_k` | `q8_0/f16` | off | 3.4188 | 3.5468 | n/c | n/c | n/c | n/c | strongest response tie cluster |
| Nemotron 120B | TQ | `tq3_k` | `q4_0/f16` | off | 3.4945 | 3.5504 | n/c | n/c | n/c | n/c | strongest row overall |
| Nemotron 120B | SQ | `sq3_k` | `sq3/f16` | off | 2.8774 | 3.0230 | n/c | n/c | n/c | n/c | weakest row in batch |
| Nemotron 120B | RQ | `planar3_k` | `planar3/f16` | off | 2.9640 | 3.1289 | n/c | n/c | n/c | n/c | below baseline prompt |
| Nemotron 120B | RQ | `iso3_k` | `iso3/f16` | off | 2.9020 | 3.1125 | n/c | n/c | n/c | n/c | below baseline prompt |

Review note:

- Nemotron was the cleanest example in this batch where TurboQuant-style rows clearly beat both SQ and RQ in the off-only rerun.
- That matters because it shows the April 10 ported suite is not just reproducing a single family bias across all models.

## Mistral Small 4 119B Ported Final Rerun

Metadata:

- model type: `mistral4 ?B Q3_K - Medium`
- model size: `50.81 GiB`
- params: `118.97B`
- offload depth: `ngl=10`

| Model | Runtime | Variant | K/V | GB | Prompt TPS | Response TPS | Peak VRAM MiB | PPL | Avg W | TTFT | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Mistral Small 4 119B | Stock | baseline | `f16/f16` | off | 4.6660 | 4.7085 | n/c | n/c | n/c | n/c | stock baseline |
| Mistral Small 4 119B | TQ | `tq1_k` | `q8_0/f16` | off | 10.0010 | 10.6750 | n/c | n/c | n/c | n/c | strongest response row |
| Mistral Small 4 119B | TQ | `tq3_k` | `q4_0/f16` | off | 10.4842 | 10.6005 | n/c | n/c | n/c | n/c | strongest prompt among TQ |
| Mistral Small 4 119B | SQ | `sq3_k` | `sq3/f16` | off | 10.0729 | 9.3975 | n/c | n/c | n/c | n/c | strong prompt, weaker gen than TQ |
| Mistral Small 4 119B | RQ | `planar3_k` | `planar3/f16` | off | 8.1761 | 8.2309 | n/c | n/c | n/c | n/c | clearly ahead of baseline |
| Mistral Small 4 119B | RQ | `iso3_k` | `iso3/f16` | off | 8.1361 | 7.6733 | n/c | n/c | n/c | n/c | slower than planar3 and all TQ rows |

Review note:

- Mistral was the most dramatic uplift in the ported rerun set.
- Both TQ rows more than doubled baseline throughput, while SQ was also clearly better than stock.
- RotorQuant still improved heavily over stock, but it was not the leader on this model.

## Cross-Model Findings

The April 10 rerun does not support a single universal winner.

Instead, the more defensible conclusions are:

1. RotorQuant remained strongest on prompt throughput for Qwen3.5-9B and Gemma 4 31B.
2. RotorQuant stayed competitive on Qwen3.5-27B, with `iso3_k` the strongest non-baseline row.
3. Qwen3.5-122B in the off-only ported rerun did not replicate the earlier GreenBoost-on `iso3` win; the April 10 rerun favored `tq3_k` on prompt throughput and `planar3_k` on response throughput.
4. Nemotron 120B and Mistral 119B both favored TurboQuant-family rows over SQ and RQ in this batch.
5. SpectralQuant `sq3_k` remained runnable across all six model families in this rerun, but it was only occasionally near the front of the pack and was not the dominant family in this off-only suite.

## Later April 10 Gapfill Results

After the off-only rerun was published, the study continued with four useful follow-on phases:

1. a full GreenBoost-on ported throughput rerun across all six model families;
2. a fresh ctx=512 perplexity / peak-VRAM pass;
3. a 2048-context quality sweep plus variant gapfills;
4. an April 11 integrity audit and targeted backfills that corrected several previously over-optimistic summaries.

These later results materially improve completeness, but they also invalidate several previously published 2048 baseline PPL claims.

### GreenBoost-On Throughput Summary

All six GreenBoost-on ported throughput runs completed successfully.

| Model | Baseline Prompt TPS | Baseline Response TPS | Best Prompt Variant | Best Prompt TPS | Best Response Variant | Best Response TPS | Takeaway |
|---|---:|---:|---|---:|---|---:|---|
| Qwen 9B | 2930.0998 | 68.8220 | baseline | 2930.0998 | `sq3_k` | 69.8865 | GreenBoost-on erased the earlier RotorQuant prompt lead; stock stayed best on prompt while SQ3 edged response throughput. |
| Qwen 27B | 87.9964 | 2.2811 | baseline | 87.9964 | baseline | 2.2811 | The on-run favored stock outright; all compressed variants stayed below baseline on both axes. |
| Qwen 122B | 1.9445 | 2.6265 | `sq3_k` | 4.1340 | `planar3_k` | 4.2550 | GreenBoost-on substantially improved the compressed rows relative to stock on this model. |
| Gemma 4 31B | 43.4743 | 1.1079 | `planar3_k` | 43.8766 | `tq1_k` | 1.1098 | Gemma remained tightly clustered, with only modest separation between families. |
| Nemotron 120B | 3.1263 | 3.1458 | `sq3_k` | 3.1825 | `iso3_k` | 3.1647 | Nemotron tightened dramatically under GreenBoost-on; family differences became small. |
| Mistral 119B | 6.7776 | 7.1599 | `sq3_k` | 9.5384 | `tq3_k` | 9.7294 | Mistral again showed large gains from compressed K-only variants under the ported-on setup. |

Review note:

- The GreenBoost-on rerun does not merely replay the GreenBoost-off story.
- Qwen 122B was the clearest same-day reversal: stock was weakest on both axes, while `sq3_k`, `tq3_k`, and `planar3_k` all moved into the lead cluster.
- Qwen 27B moved in the opposite direction, with stock staying clearly ahead of all compressed rows.

### April 11 Integrity Audit

An audit on April 11 found that the original monitored-perplexity collector accepted the first chunk PPL value as final even when later chunks went `nan`.

That means several previously summarized baseline rows are not valid PPL measurements.

Affected rows reclassified from usable to invalid:

- Qwen 27B baseline, ctx 2048
- Qwen 122B baseline, ctx 2048
- Gemma 4 31B baseline, ctx 2048
- Nemotron 120B baseline, ctx 2048
- Qwen 27B baseline, ctx 8192 / 16384 / 40960
- Gemma 4 31B baseline, ctx 8192 / 16384 / 40960

At the same time, targeted backfills improved completeness elsewhere:

- Qwen 122B `planar3_k` is now filled at ctx 512 and ctx 2048.
- Mistral 119B `tq1_k`, `tq3_k`, and `sq3_k` are now filled at ctx 512 and ctx 2048.
- The current audited 512/2048 quality matrix reports 72 expected rows, 0 missing rows, 62 `ok` rows, and 10 explicit `failed` rows.

### Fresh ctx=512 Perplexity And Peak-VRAM Summary

The ctx=512 quality matrix is now complete in the sense that there are no silent gaps left. Rows are either measured or explicitly failed.

| Model | Best PPL Variant | Best PPL | Lowest Peak-VRAM Variant | Lowest Peak VRAM MiB | Remaining Gap |
|---|---|---:|---|---:|---|
| Qwen 9B | `tq1_k` | 4.0536 | baseline | 6566 | none |
| Qwen 27B | `tq1_k` | 3.5412 | baseline | 11432 | none |
| Qwen 122B | `tq1_k` | 3.7452 | `iso3_k` | 10568 | none |
| Gemma 4 31B | baseline | 119.9030 | baseline | 7820 | none |
| Nemotron 120B | baseline | 4.0351 | baseline | 9430 | none |
| Mistral 119B | `tq1_k` | 32122.2728 | `tq1_k` | 7343 | baseline, `planar3_k`, and `iso3_k` remain explicit failures |

Review note:

- The fresh quality data remained unusually stable across families on the Qwen models, with very small absolute PPL spreads despite throughput differences.
- Gemma and Nemotron both favored baseline on PPL and VRAM, even though some compressed variants were faster in the throughput suites.
- Mistral 119B is no longer a missing-data problem at ctx 512, but it is still a quality blocker story: the surviving TQ/SQ rows complete with pathologically high perplexity, while baseline and both RotorQuant rows fail.

#### Full ctx=512 PPL / Max-VRAM Rows

##### Qwen 9B ctx=512

| Variant | Status | PPL | Max VRAM MiB | Delta VRAM MiB |
|---|---|---:|---:|---:|
| baseline | ok | 4.0681 | 6566 | 5580 |
| `tq1_k` | ok | 4.0536 | 6592 | 5606 |
| `tq3_k` | ok | 4.0754 | 6590 | 5604 |
| `sq3_k` | ok | 4.0754 | 6590 | 5604 |
| `planar3_k` | ok | 4.0627 | 6570 | 5584 |
| `iso3_k` | ok | 4.0627 | 6590 | 5604 |

##### Qwen 27B ctx=512

| Variant | Status | PPL | Max VRAM MiB | Delta VRAM MiB |
|---|---|---:|---:|---:|
| baseline | ok | 3.5447 | 11432 | 10446 |
| `tq1_k` | ok | 3.5412 | 11450 | 10454 |
| `tq3_k` | ok | 3.5513 | 11448 | 10462 |
| `sq3_k` | ok | 3.5513 | 11448 | 10462 |
| `planar3_k` | ok | 3.5461 | 11444 | 10458 |
| `iso3_k` | ok | 3.5461 | 11444 | 10458 |

##### Qwen 122B ctx=512

| Variant | Status | PPL | Max VRAM MiB | Delta VRAM MiB |
|---|---|---:|---:|---:|
| baseline | ok | 3.7456 | 10580 | 9594 |
| `tq1_k` | ok | 3.7452 | 10582 | 9575 |
| `tq3_k` | ok | 3.7466 | 10580 | 9594 |
| `sq3_k` | ok | 3.7466 | 10580 | 9573 |
| `planar3_k` | ok | 3.7543 | 10477 | 9561 |
| `iso3_k` | ok | 3.7543 | 10568 | 9582 |

##### Gemma 4 31B ctx=512

| Variant | Status | PPL | Max VRAM MiB | Delta VRAM MiB |
|---|---|---:|---:|---:|
| baseline | ok | 119.9030 | 7820 | 6834 |
| `tq1_k` | ok | 121.6342 | 7854 | 6868 |
| `tq3_k` | ok | 124.4262 | 7854 | 6868 |
| `sq3_k` | ok | 124.4262 | 7874 | 6888 |
| `planar3_k` | ok | 119.9030 | 7842 | 6856 |
| `iso3_k` | ok | 119.9030 | 7842 | 6856 |

##### Nemotron 120B ctx=512

| Variant | Status | PPL | Max VRAM MiB | Delta VRAM MiB |
|---|---|---:|---:|---:|
| baseline | ok | 4.0351 | 9430 | 8444 |
| `tq1_k` | ok | 4.0457 | 9432 | 8446 |
| `tq3_k` | ok | 4.0479 | 9432 | 8446 |
| `sq3_k` | ok | 4.0479 | 9432 | 8446 |
| `planar3_k` | ok | 4.0351 | 9430 | 8442 |
| `iso3_k` | ok | 4.0351 | 9430 | 8442 |

##### Mistral 119B ctx=512

| Variant | Status | PPL | Max VRAM MiB | Delta VRAM MiB |
|---|---|---:|---:|---:|
| baseline | failed | blocked | 7215 | 6299 |
| `tq1_k` | ok | 32122.2728 | 7343 | 6427 |
| `tq3_k` | ok | 47109.3475 | 7343 | 6427 |
| `sq3_k` | ok | 47109.3475 | 7343 | 6427 |
| `planar3_k` | failed | blocked | 6087 | 5171 |
| `iso3_k` | failed | blocked | 6087 | 5171 |

### 2048-Context Update

The 2048-context matrix is now materially more complete, but also more conservative after the integrity audit.

#### Reclassified 2048 Baseline Rows

These baseline runs still provide peak-VRAM evidence, but they do not provide valid final PPL values and should not be used in quality ranking tables.

| Model | Status | PPL | Peak VRAM MiB | Delta VRAM MiB | Note |
|---|---|---:|---:|---:|---|
| Qwen 27B | failed | blocked | 6611 | 5741 | partial PPL then `nan` |
| Qwen 122B | failed | blocked | 8007 | 7137 | partial PPL then `nan` |
| Gemma 4 31B | failed | blocked | 5385 | 4515 | partial PPL then `nan` |
| Nemotron 120B | failed | blocked | 6607 | 5737 | partial PPL then `nan` |
| Mistral 119B | failed | blocked | 7979 | 7109 | null-PPL / `nan` path |

#### Audited 2048 Variant Results

| Model | Best Valid 2048 PPL Variant | Best Valid 2048 PPL | Lowest Valid 2048 Peak-VRAM Variant | Lowest Valid 2048 Peak VRAM MiB | Coverage |
|---|---|---:|---|---:|---|
| Qwen 9B | `planar3_k` / `iso3_k` | 4.2249 | `tq3_k` / `sq3_k` | 6233 | full six-variant set completed |
| Qwen 27B | `planar3_k` / `iso3_k` | 4.0129 | `tq3_k` | 6619 | baseline invalid; five valid variant rows remain |
| Qwen 122B | `planar3_k` / `iso3_k` | 3.4578 | `planar3_k` | 8067 | all non-baseline variants now filled |
| Gemma 4 31B | `planar3_k` / `iso3_k` | 96.0670 | `planar3_k` | 5530 | baseline invalid; five valid variant rows remain |
| Nemotron 120B | `tq1_k` | 3.8195 | `tq1_k` | 6749 | full non-baseline set completed |
| Mistral 119B | `tq1_k` | 93376.2709 | `tq3_k` / `sq3_k` | 7615 | TQ/SQ rows completed, baseline and RotorQuant rows failed |

Review note:

- Qwen 9B remains the cleanest 2048-quality family in this ported study.
- Qwen 122B no longer has a `planar3_k` coverage hole at 2048.
- Nemotron now has a complete non-baseline 2048 variant set.
- Mistral 119B at 2048 is no longer a missing-data story, but the surviving TQ/SQ rows still show extremely poor quality and the baseline / RotorQuant rows remain blocked.

### April 11 Large-Context Stress Update

The April 11 `8192 / 16384 / 40960` response sweep remains useful, but only as stress and scaling evidence.

After the integrity audit:

- Qwen 9B baseline remains the only fully valid baseline curve at all three large-context checkpoints.
- Qwen 27B baseline is invalid at `8192`, `16384`, and `40960`, even after a reduced-batch `8192` retry.
- Gemma 4 31B baseline is invalid at `8192`, `16384`, and `40960`, even after a reduced-batch `8192` retry.
- Valid 40960 quantized-KV checks still remain for Qwen 9B `tq1_k`, Qwen 27B `tq1_k`, and Gemma 4 31B `tq1_k`.

This means the current 40k-class evidence should be treated as a stress / scaling appendix, not as headline quality proof.

## Publication Guidance

This file should be read as the April 10 ported-suite update, not as a full replacement for the main repository summary.

Recommended interpretation:

- keep the main README as the higher-level cross-day narrative;
- use this note as the detailed publication record of the April 10 rerun and the same-day gapfill batches that followed it;
- cite these results specifically when discussing the ported-tree validation batch and the newly added Gemma / Nemotron / Mistral model coverage.

## Source Files

Primary driver and summary:

- `spectralquant_study/scripts/run_ported_benchmark_suite_20260410.sh`
- `spectralquant_study/runs/20260410_ported_suite_summary.tsv`
- `spectralquant_study/runs/20260410_ported_greenboost_on_suite_summary.tsv`
- `spectralquant_study/runs/20260410_ported_greenboost_on_ppl_combined_summary.tsv`
- `spectralquant_study/runs/20260410_targeted_quality_gapfill_summary.tsv`
- `spectralquant_study/runs/20260410_full_quality_matrix_completeness.md`
- `spectralquant_study/runs/20260410_ranked_quality_summary.md`
- `spectralquant_study/runs/20260410_ported_long_context_sweep_summary.tsv`
- `spectralquant_study/runs/20260410_ported_long_context_gapfill_variants_summary.tsv`
- `spectralquant_study/runs/20260411_stats_integrity_audit.md`
- `spectralquant_study/runs/20260411_large_context_response_summary.tsv`
- `spectralquant_study/runs/20260411_large_context_response_report.md`

Primary run roots:

- `spectralquant_study/runs/20260410T072755Z_qwen9b-k-only-all-families-ported-off`
- `spectralquant_study/runs/20260410T093144Z_qwen27b-k-only-all-families-ported-off-rerun`
- `spectralquant_study/runs/20260410T073749Z_qwen122b-k-only-all-families-ported-off`
- `spectralquant_study/runs/20260410T074043Z_gemma4-31b-k-only-all-families-ported-off`
- `spectralquant_study/runs/20260410T081405Z_nemotron120b-k-only-all-families-ported-off`
- `spectralquant_study/runs/20260410T115510Z_mistral119b-k-only-all-families-ported-off-final`