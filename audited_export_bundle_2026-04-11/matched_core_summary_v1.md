# Matched Core Summary v1

This table is a derived publication view that joins core quality rows to throughput rows from a separate but equivalent April 10 ported-on experiment cohort.

## Merge Rule

- join key: same `model`, `variant`, and `greenboost` state
- throughput source: `throughput_ported_on_final` rows from `master_table_v1.tsv`
- quality source: `core_matrix_v1_ctx512` and `core_matrix_v1_ctx2048` rows from `master_table_v1.tsv`
- provenance: throughput and quality remain separate run roots in the output so the join can be audited

## Equivalence Tiers

- `strong`: same model / variant / GreenBoost state and the quality row is the `ctx=512` short-context comparison tier
- `usable_with_ctx_caveat`: same model / variant / GreenBoost state, but the quality row is the `ctx=2048` medium-context checkpoint while throughput still comes from the final ported-on benchmark envelope

## Coverage

- matched rows: 71
- `strong` rows: 36
- `usable_with_ctx_caveat` rows: 35
- rows whose quality status remains blocker / missing: 11

## Use Rule

- use this table for side-by-side scanning of throughput plus quality evidence within the same runtime family and April 10 publication cohort
- do not read it as a single-run dense matrix; it is a documented join across multiple experiments
- when a headline depends on exact context behavior, prefer the original source rows in `master_table_v1.tsv` and the audit notes
