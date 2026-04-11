# Mistral Blocker Note

## Current Status

Mistral 119B is a blocker cluster, not a missing-data problem.

## Blocked Rows In Core Matrix v1

- baseline at `ctx=512`
- `planar3_k` at `ctx=512`
- `iso3_k` at `ctx=512`
- baseline at `ctx=2048`
- `planar3_k` at `ctx=2048`
- `iso3_k` at `ctx=2048`

## Existing Investigation Evidence

- upstream baseline probe: `runs/20260410_mistral_upstream_probe/upstream_baseline_ngl4_b128.summary.json`
- GreenBoost tree with GreenBoost disabled: `runs/20260410_mistral_greenboostoff_probe/gbtree_off_baseline_ngl4_b128.summary.json`
- reduced-pressure ported probes: `runs/20260410_mistral_ppl_probe/`

## Current Interpretation

- baseline `null` PPL reproduces outside GreenBoost-on mode;
- the issue is therefore not cleanly attributable to GreenBoost injection alone;
- `tq1_k`, `tq3_k`, and `sq3_k` can complete under reduced-pressure settings, but baseline / planar / iso remain blocked;
- this should be reported as an evaluator, model-path, or runtime-family blocker under current settings.

Observed failure signature in both upstream and GreenBoost-disabled baseline probes:

- all chunk outputs are `nan` from the start (`[1]nan,[2]nan,...`)
- the run ends with `Unexpected negative standard deviation of log(prob)`

Observed reduced-pressure survivor rows:

- `ctx=512`: `tq1_k`, `tq3_k`, `sq3_k` succeeded via `runs/20260411T015013Z_mistral119b-tq1_k-ctx512`, `runs/20260411T015417Z_mistral119b-tq3_k-ctx512`, and `runs/20260411T015825Z_mistral119b-sq3_k-ctx512`
- `ctx=2048`: `tq1_k`, `tq3_k`, `sq3_k` succeeded via `runs/20260411T004743Z_mistral119b-tq1-longctx-2048`, `runs/20260411T010111Z_mistral119b-tq3-longctx-2048`, and `runs/20260411T011443Z_mistral119b-sq3-longctx-2048`

## Publication Rule

Do not hide Mistral failures by omission. Keep them in a visible blocker section and exclude them from headline comparison tables.
