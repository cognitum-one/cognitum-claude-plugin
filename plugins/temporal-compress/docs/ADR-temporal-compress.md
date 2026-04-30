# ADR: Temporal Compress as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `temporal-compress`
**Category**: signal

## Context

A seed running multiple cogs at 1-30 second cadences produces tens of
megabytes per day of feature-frame history. Most of that data is
redundant — adjacent frames in a quiet period look almost identical.
Trace-replay debugging and slow-trend cogs only need *summary*
information from old frames, not the full stream. Temporal-compress is
the pyramidal-summarisation cog that lets the seed keep weeks of
history in the same RAM and disk budget as a few hours of raw stream.

Signal-processing approach: a fixed-ratio pyramid. The most recent N
seconds are stored at full resolution; the next tier is 8:1 averaged
with min/max preserved (so spikes survive); the next tier is another
8:1 reduction; and so on. Reduction uses a piecewise-aggregate
approximation (PAA) variant that keeps mean, variance, min, and max
per block, which is enough for downstream replay and threshold queries
to behave close to the raw stream.

## Decision

Standalone armhf binary, reads `cog-sensor-sources` at 10 s default
"compression cycle" cadence. Pipeline: append to tier-0 ring → when
tier-0 fills, fold the oldest 8 frames into a tier-1 block (mean, var,
min, max, count, ts_start, ts_end) → cascade. Output JSON reports
current tier sizes, total compression ratio, oldest sample timestamp,
and per-tier byte usage. Disk persistence is optional via `--persist`.

As Claude Code plugin: `/temporal-compress` wraps cog endpoints.

## Consequences

### Positive
- 64x to 512x effective compression for steady streams without
  losing spike information.
- Tier-0 still gives downstream cogs the full-resolution recent
  window they need.
- Ratio and oldest-timestamp metrics make capacity planning explicit.

### Negative
- Random access to old data is block-resolution only; per-frame
  recovery beyond tier-0 is impossible by design.
- Pyramidal mean smooths fast oscillations in distant tiers.

### Neutral
- The 8:1 ratio per tier is a tunable but works well as default.

## Alternatives considered
- **Wavelet compression.** Rejected: bigger binary, marginal gain at
  these signal-to-noise ratios.
- **Lossless gzip of the raw stream.** Rejected: still grows linearly
  with time, defeating the purpose.

## Plugin invocation
- `/temporal-compress` install, start, tail logs
- `/temporal-compress --once`
- `/temporal-compress --console "--interval 5"`
- `/temporal-compress --stop`
- `/temporal-compress --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB (multi-tier rings, fixed total size)
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/temporal-compress/`
- ADR-001 foundational
- Related cogs: `time-crystal`, `sparse-recovery`
