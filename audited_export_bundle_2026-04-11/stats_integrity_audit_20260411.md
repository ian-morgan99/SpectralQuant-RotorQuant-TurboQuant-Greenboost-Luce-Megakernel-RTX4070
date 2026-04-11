# Stats Integrity Audit

This audit was performed on 2026-04-11 after reviewing all gathered PPL summaries and raw logs.

## Root Cause

`scripts/run_monitored_perplexity.sh` previously accepted the first chunk-level PPL value as `final_ppl` even when the rest of the run produced `nan` and ended with `Unexpected negative standard deviation of log(prob)`.

The collector has now been fixed so any run with that failure signature leaves `final_ppl` unset.

## Newly Reclassified Invalid Rows

These rows were previously summarized as `ok` but are not valid PPL measurements:

- Qwen 27B baseline, ctx 2048: `runs/20260410T211100Z_qwen27b-baseline-longctx-2048`
- Qwen 122B baseline, ctx 2048: `runs/20260410T202629Z_qwen122b-baseline-longctx-2048`
- Gemma 4 31B baseline, ctx 2048: `runs/20260410T203108Z_gemma4-31b-baseline-longctx-2048`
- Nemotron 120B baseline, ctx 2048: `runs/20260410T204151Z_nemotron120b-baseline-longctx-2048`
- Qwen 27B baseline, ctx 8192: `runs/20260411T021350Z_qwen27b-baseline-ctx8192`
- Qwen 27B baseline, ctx 16384: `runs/20260411T032127Z_qwen27b-baseline-ctx16384`
- Qwen 27B baseline, ctx 40960: `runs/20260411T042719Z_qwen27b-baseline-ctx40960`
- Gemma 4 31B baseline, ctx 8192: `runs/20260411T023310Z_gemma4-31b-baseline-ctx8192`
- Gemma 4 31B baseline, ctx 16384: `runs/20260411T034014Z_gemma4-31b-baseline-ctx16384`
- Gemma 4 31B baseline, ctx 40960: `runs/20260411T044625Z_gemma4-31b-baseline-ctx40960`

## New Tests Run During Audit

To check whether these were just collector artifacts or recoverable with lower-pressure settings, two fresh retries were run with reduced batch size.

- Qwen 27B baseline, ctx 8192, batch 64: `runs/20260411T064415Z_qwen27b-baseline-ctx8192-b64-clean`
  - Result: still invalid
  - Summary: `final_ppl: null`
  - Log pattern: `[1]11.3517,[2]nan,...` followed by `Unexpected negative standard deviation of log(prob)`
- Gemma 4 31B baseline, ctx 8192, batch 64: `runs/20260411T072056Z_gemma4-31b-baseline-ctx8192-b64-clean`
  - Result: still invalid
  - Summary: `final_ppl: null`
  - Log pattern: `[1]3.6223,[2]nan,...` followed by `Unexpected negative standard deviation of log(prob)`

## Still Valid Large-Context Rows

These rows remain valid after audit:

- Qwen 9B baseline: 8192, 16384, 40960
- Qwen 9B `tq1_k`: 40960
- Qwen 27B `tq1_k`: 40960
- Gemma 4 31B `tq1_k`: 40960

## Summary Impact

- `20260410_full_quality_matrix_completeness.md` now reports 62 `ok` rows and 10 `failed` rows for the 512/2048 matrix.
- `20260411_large_context_response_summary.tsv` now marks the six invalid Qwen 27B and Gemma baseline rows as `failed`.
- `20260411_large_context_response_report.md` now treats only the Qwen 9B baseline curve as valid and documents the Qwen 27B and Gemma baseline blockers.
