# Methods and Limitations

## How To Read This Data

1. Cross-family absolute PPL is not a safe ranking metric.
2. `ctx=2048` is a medium-context checkpoint, not definitive large-context coverage.
3. Repeated-corpus `40960` runs are stress tests, not final quality proof.
4. Failed rows are blockers, not zeros.

## GreenBoost On / Off Interpretation

- Throughput comparisons are directional and useful for runtime-family ranking.
- Current throughput evidence is mostly low-repeat, so treat it as `B`-grade directional evidence, not hard statistical proof.

## Quality Evidence Boundaries

- `ctx=512` is the main comparison tier for within-model PPL and peak-VRAM comparison across variants.
- `ctx=2048` extends that evidence into medium-context behavior.
- `8192`, `16384`, and `40960` should remain in a separate large-context track.

## Large-Context Caveat

The April 11 large-context sweep used a repeated text corpus. That makes the run useful for:

- completion and stability checks;
- peak VRAM growth;
- functional validation of quantized-KV paths;
- stress and scaling interpretation.

It does not make the resulting absolute PPL values strong real-world long-context quality evidence.

## Collector Integrity Note

On 2026-04-11, `run_monitored_perplexity.sh` was fixed so that runs ending in `Unexpected negative standard deviation of log(prob)` and `nan` chunk outputs no longer emit a false `final_ppl`.

The audit result is documented at:

- `stats_integrity_audit_20260411.md`

## Confidence Grades Used In `master_table_v1.tsv`

- `B`: final throughput reruns and valid core-matrix rows
- `C`: stress-only or anomaly-under-review rows
- `F`: blocked, failed, or invalid rows
