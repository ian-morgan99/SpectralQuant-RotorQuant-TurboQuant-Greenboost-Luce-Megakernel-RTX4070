# Large-Context Stress Report

This document is the publication-facing wrapper for the April 11 large-context work.

## Scope

- contexts: `8192`, `16384`, `40960`
- models targeted: Qwen 9B, Qwen 27B, Gemma 4 31B
- variants: baseline plus `tq1_k` at `40960`

## Interpretation Rule

These rows are stress and scaling evidence. They should not be merged into the core quality matrix without explicit corpus caveats.

## Validity Summary

- Qwen 9B baseline remains the only fully valid baseline curve in the repeated-corpus `8192` to `40960` sweep.
- Qwen 27B and Gemma 4 31B baseline rows were reclassified after audit because they emitted a first chunk PPL and then went `nan`.
- Qwen 27B `tq1_k` at `40960` completed cleanly.
- Gemma 4 31B `tq1_k` at `40960` completed cleanly.

## Primary Sources

- `runs/20260411_large_context_response_summary.tsv`
- `runs/20260411_large_context_response_report.md`
- `stats_integrity_audit_20260411.md`

## Context Ladder

- `2048`: medium-context checkpoint
- `8192`: first credible long-context tier
- `16384`: stronger long-context validation tier
- `40960`: stress and envelope tier
