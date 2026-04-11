# Qwen 27B Non-Repeated Anomaly Pack

This report is populated from the non-repeated corpus reruns executed via:

- `scripts/run_qwen27b_nonrepeated_anomaly_pack_20260411.sh`

Primary summary artifact:

- `runs/20260411_qwen27b_nonrepeated_anomaly_pack_summary.tsv`

## Purpose

The goal of this pack is to test whether the previously anomalous Qwen 27B large-context behavior is:

- repeated-corpus induced;
- evaluator induced;
- or a real model/runtime effect.

## Target Rows

- baseline at `2048`, `8192`, `16384`
- `tq1_k` at `2048`, `8192`, `16384`

## Corpus

- non-repeated corpus: Wikitext-2 test split (`wiki.test.raw`)

## Results

| Variant | ctx 2048 | ctx 8192 | ctx 16384 | Interpretation |
|---|---:|---:|---:|---|
| baseline | failed | failed | failed | same failure pattern at all tested contexts |
| `tq1_k` | 6.6734 | 6.9558 | 6.4163 | completed cleanly at all tested contexts |

## Peak VRAM Summary

| Variant | ctx 2048 peak MiB | ctx 8192 peak MiB | ctx 16384 peak MiB |
|---|---:|---:|---:|
| baseline | 6856 | 6780 | 6940 |
| `tq1_k` | 6680 | 6780 | 6980 |

## Failure Signature

The non-repeated corpus did not remove the baseline issue. The baseline rows still show a first chunk value followed by `nan` for the remaining chunks and terminate with:

- `Unexpected negative standard deviation of log(prob)`

Example baseline evidence:

- `runs/20260411T085918Z_qwen27b-baseline-nonrepeated-ctx2048/raw/perplexity/baseline.log`

Example successful `tq1_k` evidence:

- `runs/20260411T093946Z_qwen27b-tq1_k-nonrepeated-ctx2048/raw/perplexity/tq1_k.log`

## Interpretation

This result tightens the anomaly assessment substantially.

- The odd Qwen 27B behavior is not just a repeated-corpus artifact.
- On the tested Wikitext-2 non-repeated corpus, the baseline path is unstable at `2048`, `8192`, and `16384`.
- The `tq1_k` path remains functional and internally consistent across the same contexts.

Current grading:

- baseline rows in this anomaly pack: `F`
- `tq1_k` rows in this anomaly pack: `C`

They remain `C` rather than `B` because the pack is still a focused anomaly investigation rather than a replicated publication-grade matrix.
