# Core Matrix v1

`Core Matrix v1` is the frozen comparison scope for the main April 10 benchmark pass.

## Frozen Scope

- Models: Qwen 9B, Qwen 27B, Qwen 122B, Gemma 4 31B, Nemotron 120B, Mistral 119B
- Contexts: 512 and 2048
- Variants: baseline, `tq1_k`, `tq3_k`, `sq3_k`, `planar3_k`, `iso3_k`

## Intent

This matrix is the main evidence set for the current publication layer. It should not be rerun wholesale before anomaly and blocker work is resolved.

## Included Evidence Roots

- `ctx=512` quality source: `runs/20260410_ported_greenboost_on_ppl_combined_summary.tsv`
- `ctx=2048` baseline source: `runs/20260410_ported_long_context_sweep_summary.tsv`
- `ctx=2048` variant source: `runs/20260410_ported_long_context_gapfill_variants_summary.tsv`
- targeted backfills: `runs/20260410_targeted_quality_gapfill_summary.tsv`
- completeness note: `runs/20260410_full_quality_matrix_completeness.md`

## Current Status Interpretation

- Silent gaps are no longer the main problem.
- The remaining issues are explicit blockers and evidence-classification limits.
- The current valid core rows are publication-usable with moderate caution.

## Visible Blocker Section

Current blocker rows that must stay visible in publication material:

- Mistral 119B baseline at `ctx=512`
- Mistral 119B `planar3_k` at `ctx=512`
- Mistral 119B `iso3_k` at `ctx=512`
- Mistral 119B baseline at `ctx=2048`
- Mistral 119B `planar3_k` at `ctx=2048`
- Mistral 119B `iso3_k` at `ctx=2048`
- Qwen 27B baseline at `ctx=2048`
- Qwen 122B baseline at `ctx=2048`
- Gemma 4 31B baseline at `ctx=2048`
- Nemotron 120B baseline at `ctx=2048`

The last four rows were reclassified from `ok` to `failed` after the 2026-04-11 integrity audit found partial-`nan` baseline runs had been incorrectly accepted by the old collector. See `stats_integrity_audit_20260411.md`.

## Throughput Layer Selection

Use only final usable throughput roots:

- GreenBoost off final roots: Qwen 9B ported-off, Qwen 27B replication rerun, Qwen 122B ported-off, Gemma 4 31B ported-off, Nemotron 120B ported-off, Mistral 119B final ported-off rerun
- GreenBoost on final roots: the six `20260410T14...` to `20260410T15...` ported-on family runs

First-pass failed throughput roots should not appear in publication comparison tables except in status notes.
