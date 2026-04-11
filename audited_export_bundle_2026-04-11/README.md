# Audited Export Bundle, 2026-04-11

This directory is the publication-facing layer for the April 10 to April 11 benchmark pass.

Use these files in order:

1. `README.md`: executive summary and reading order.
2. `core_matrix_v1.md`: the frozen 512 and 2048 comparison matrix scope.
3. `methods_and_limitations.md`: how to interpret the evidence and its limits.
4. `stats_integrity_audit_20260411.md`: the collector-integrity correction that reclassified invalid rows.
5. `large_context_stress_report.md`: 8192 to 40960 stress-only findings.
6. `mistral_blocker_note.md`: current blocker interpretation for Mistral.
7. `qwen27b_anomaly_pack_nonrepeated_20260411.md`: anomaly rerun report on a non-repeated corpus.
8. `master_table_v1.tsv`: machine-readable source of truth for curated rows.
9. `matched_core_summary_v1.md` and `matched_core_summary_v1.tsv`: derived joined view for core quality rows plus equivalent April 10 ported-on throughput rows.

## Master Table Shape

`master_table_v1.tsv` is a sparse union table, not a single fully dense benchmark matrix.

- `throughput_*` rows carry `prompt_tps` and `response_tps`.
- `core_matrix_*`, `large_context_stress`, and `qwen27b_nonrepeated_anomaly_pack` rows carry PPL / VRAM evidence instead.
- Blank TPS fields on non-throughput rows mean `not measured in that suite`, not `missing from upload`.
- Reusing throughput from a different envelope was avoided on purpose, because that would make the row look denser while mixing incompatible measurement contexts.

`matched_core_summary_v1.tsv` is the narrower convenience layer for readers who do want a side-by-side throughput-plus-quality view.

- It is a documented join across multiple experiments, not a single-run matrix.
- It only joins `core_matrix_v1_*` quality rows to `throughput_ported_on_final` rows when `model`, `variant`, and `greenboost` match.
- Each joined row keeps both the throughput and quality run roots so provenance remains visible.
- `ctx=512` rows are labeled `strong`; `ctx=2048` rows are labeled `usable_with_ctx_caveat`.

## Headline Findings

- The 512 and 2048 matrix is broad enough to freeze as `Core Matrix v1`, but some previously accepted baseline PPL rows were reclassified after a collector-integrity audit.
- Throughput evidence is useful and broad, but only the final usable rerun roots should be treated as publication rows.
- The 8192 to 40960 April 11 sweep is useful as stress and scaling evidence, not as headline real-world quality proof.
- The Qwen 27B non-repeated anomaly pack shows the baseline failure persists on Wikitext-2 at `2048`, `8192`, and `16384`, while `tq1_k` completes cleanly at all three contexts.
- Mistral remains a blocker cluster and should be surfaced explicitly rather than omitted.

## Confidence Model

- `A`: repeated and internally corroborated headline evidence.
- `B`: usable publication evidence with moderate caution.
- `C`: completed but anomalous or strongly caveated.
- `F`: blocked, failed, or invalid for comparison use.

Current practical grading:

- April 10 final throughput reruns: `B`
- April 10 `ctx=512` core quality rows: `B`
- April 10 `ctx=2048` core quality rows: `B` for valid rows, `F` for blocker rows
- April 11 repeated-corpus large-context rows: `C` for valid rows, `F` for invalid rows
- Mistral blocker rows: `F`

## How To Read This Export Set

- Do not use cross-family absolute PPL as a ranking metric.
- Do not treat `ctx=2048` alone as definitive large-context coverage.
- Do not treat repeated-corpus `40960` results as final quality proof.
- Treat failed rows as blockers, not zeros.

## Artifact Path Note

Paths beginning with `runs/` and `scripts/` are provenance references from the original study workspace. They are kept here as compact source labels; the raw run directories are not mirrored in this publication repo.
