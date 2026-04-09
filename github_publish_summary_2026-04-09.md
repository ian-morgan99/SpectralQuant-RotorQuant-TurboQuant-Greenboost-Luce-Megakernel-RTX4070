# Qwen3.5-9B Runtime Family Comparison and RTX 4070 DVFS Notes

This document is a single-file publication summary for the current Qwen3.5-9B runtime-family comparison work and the follow-up RTX 4070 power-limit sweep.

## Scope

This note covers three related questions:

1. How the active TurboQuant / SpectralQuant runtime family compares against the first runnable RotorQuant-family fork on Qwen3.5-9B.
2. Whether the same all-family comparison can currently be extended to Qwen3.5-122B on this RTX 4070 host.
3. Whether RTX 4070 DVFS power limiting materially changes the practical performance or cost-efficiency picture.

## Test Environment

- Host GPU: NVIDIA GeForce RTX 4070, 12 GB VRAM
- Primary 9B model: `Qwen3.5-9B-Q4_K_M.gguf`
- 122B model used for reduced-envelope probing: `Qwen3.5-122B-A10B-IQ3_S-00001-of-00002.gguf`
- Primary benchmark tool: `llama-bench`
- Main runtime binaries:
  - stock `llama.cpp`
  - active TurboQuant / SpectralQuant path
  - RotorQuant-family fork: `llama-cpp-rotorquant-llama`
- GreenBoost-on mode used:
  - `LD_PRELOAD=/usr/local/lib/libgreenboost_cuda.so`
  - `GREENBOOST_ACTIVE=1`

## Methods

### 1. 9B runtime-family comparison

The main 9B comparison used a matched benchmark envelope across the runtime families.

Common benchmark settings:

- `ngl=99`
- `p=512`
- `n=128`
- `r=2`
- `t=6`
- `b=512`
- `ub=512`
- `fa=0`

Compared variants:

- `f16/f16`
- `q8_0/f16`
- `q4_0/f16`
- `sq3/f16`
- `planar3/f16`
- `iso3/f16`

Two runtime modes were measured:

- GreenBoost off
- GreenBoost on

Primary 9B run artifacts:

- GreenBoost off:
  - `/home/ian/nvidia_greenboost/spectralquant_study/runs/20260409T155221Z_qwen9b-k-only-all-families-off`
- GreenBoost on:
  - `/home/ian/nvidia_greenboost/spectralquant_study/runs/20260409T155508Z_qwen9b-k-only-all-families-on`

Important caveat:

- this is an operational runtime-family comparison across multiple llama.cpp-family binaries, not a single-source-tree patch-only A/B.

### 2. 122B reduced-envelope probing

The original goal was to extend the family comparison to Qwen3.5-122B, but the current host could not support a full all-family table under the same envelope.

Reduced probing showed:

- `planar3/f16` is not currently viable on this host for the 122B model, including `ngl=0`
- `iso3/f16` is loadable at `ngl=10`
- `iso3/f16` fails by `ngl=12`

That means the current 122B state is still a reduced-envelope feasibility result, not a full publishable family comparison.

### 3. RTX 4070 DVFS sweep

After the runtime-family comparison, a follow-up DVFS sweep tested whether lowering board power improved `tok/J` meaningfully on this GPU.

Power-limit range reported by `nvidia-smi` on this host:

- min: `100 W`
- max: `220 W`

Measured power points:

- `220 W`
- `200 W`
- `180 W`
- `160 W`
- `140 W`

DVFS benchmark cases:

1. stock baseline:
   - binary: stock `llama.cpp`
   - KV types: `f16/f16`
2. best current GreenBoost-on RotorQuant row:
   - binary: `llama-cpp-rotorquant-llama`
   - KV types: `iso3/f16`

Common DVFS benchmark envelope:

- `ngl=99`
- `p=512`
- `n=128`
- `r=3`
- `t=6`
- `b=512`
- `ub=512`
- `fa=0`

Each run also recorded an `nvidia-smi` one-second telemetry log for active power estimation.

## Results

### 1. 9B runtime-family comparison

#### GreenBoost off

| case | prompt t/s | gen t/s |
|---|---:|---:|
| `f16/f16` | 3154.7040 | 67.7588 |
| `q8_0/f16` | 3533.3924 | 70.3899 |
| `q4_0/f16` | 3229.4478 | 67.7758 |
| `sq3/f16` | 3214.1993 | 66.9933 |
| `planar3/f16` | 3555.3492 | 67.4522 |
| `iso3/f16` | 3245.1067 | 68.9004 |

#### GreenBoost on

| case | prompt t/s | gen t/s |
|---|---:|---:|
| `f16/f16` | 3190.3459 | 67.5688 |
| `q8_0/f16` | 3249.9968 | 68.0006 |
| `q4_0/f16` | 3175.3591 | 68.1649 |
| `sq3/f16` | 3117.2023 | 68.2459 |
| `planar3/f16` | 3223.8738 | 67.2031 |
| `iso3/f16` | 3298.7315 | 68.8726 |

### 2. RTX 4070 DVFS sweep

#### Stock baseline `f16/f16`

| power limit | pp512 t/s | tg128 t/s | active avg W | gen tok/J |
|---|---:|---:|---:|---:|
| `220 W` | 3602.59 | 76.33 | 180.89 | 0.4220 |
| `200 W` | 3614.89 | 76.37 | 191.02 | 0.3998 |
| `180 W` | 3607.36 | 76.12 | 170.04 | 0.4477 |
| `160 W` | 3529.45 | 75.73 | 148.94 | 0.5085 |
| `140 W` | 3345.35 | 74.96 | 129.70 | 0.5779 |

#### RotorQuant `iso3/f16` with GreenBoost on

| power limit | pp512 t/s | tg128 t/s | active avg W | gen tok/J |
|---|---:|---:|---:|---:|
| `220 W` | 3664.98 | 76.84 | 184.41 | 0.4167 |
| `200 W` | 3667.47 | 76.84 | 180.00 | 0.4269 |
| `180 W` | 3655.02 | 76.45 | 158.94 | 0.4810 |
| `160 W` | 3600.95 | 76.15 | 143.42 | 0.5308 |
| `140 W` | 3398.81 | 75.41 | 139.30 | 0.5414 |

## Summary Conclusions

### Runtime-family comparison

1. RotorQuant is competitive with the active TurboQuant family on Qwen3.5-9B.
2. Without GreenBoost, `planar3/f16` was the strongest prompt-path row and `q8_0/f16` was the strongest generation row.
3. With GreenBoost enabled, `iso3/f16` became the strongest overall row in the 9B matrix.
4. `sq3/f16` remained operationally viable and near baseline, but it was not the speed leader on this host.

### 122B status

1. A full 122B all-family publication table is not yet available on this RTX 4070 host.
2. The current viable RotorQuant path on 122B is a reduced-envelope `iso3/f16` comparison at approximately `ngl<=10`.
3. The 122B result should therefore be described as partial feasibility, not a complete family comparison.

### DVFS takeaway

1. RTX 4070 power limiting changed efficiency more than it changed user-visible performance.
2. `160 W` was the best practical tradeoff point:
   - very small generation-throughput loss versus higher limits,
   - only a small prompt-throughput loss,
   - materially better `tok/J`.
3. `140 W` was the strongest pure efficiency point, but it gave up more prompt throughput and is better described as an efficiency mode than a default setting.
4. The overall effect size on this 4070 is modest. DVFS is real, but it is not a major lever on this host.

## Bottom Line

For publication, the clearest concise story is:

- On this RTX 4070 host, RotorQuant-family `iso3/f16` is competitive and becomes the strongest overall 9B row when GreenBoost is enabled.
- A full 122B family comparison is not yet feasible on this host, but reduced-envelope `iso3/f16` operation is viable.
- DVFS power limiting on the RTX 4070 improves efficiency modestly, with `160 W` as the best default sweet spot and `140 W` as the maximum-efficiency mode.
- Unlike higher-draw cards such as a 3090, the 4070 does not show a dramatic practical performance change under DVFS; the benefit is mainly small `tok/J` improvement rather than a major runtime unlock.