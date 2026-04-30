# ADR: Behavioral Profiler as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `behavioral-profiler`
**Category**: security

## Context

Most security cogs key on hard signatures (a jerk z-score, a packet
fingerprint, a known-bad hash). They miss the slow-rolling anomaly:
the after-hours door open, the Tuesday-only printer-room visit that
suddenly happens on Sunday, the new device that joins the mesh and
behaves *almost* like a known one. Detecting these requires a per-site
**behavioral baseline** that the seed learns autonomously — there is no
universal model for "normal" in a private home, retail floor, or lab.

The signal-processing approach is **multivariate online clustering**:
each sample is an N-dim feature (motion variance, BLE peer count,
audio band-energy, hour-of-week, dominant cog activity). A small
streaming k-means (k=8) maintains centroids; a sample's distance to its
nearest centroid, normalized by the centroid's own dispersion, gives a
**Mahalanobis-like anomaly score**. Days are bucketed so weekday and
weekend evolve separately.

## Decision

Standalone armhf binary on seed. Reads `cog-sensor-sources` feature
stream every 5 s. Maintains 8 streaming centroids per day-bucket
(weekday/weekend × 4 time-of-day quadrants = 8). Warm-up period of
72 h before flagging anomalies. State machine:
`learning → nominal → anomaly → ALARM`. Emits JSON with
`anomaly_score`, `cluster_id`, `bucket`, `days_observed`. Packaged as
Claude Code plugin: slash command `/behavioral-profiler` wraps seed's
cog management endpoints.

## Consequences

### Positive
- Site-specific — a quiet workshop and a noisy retail floor each
  develop their own baseline without operator tuning.
- Catches the "almost-normal" anomalies that signature cogs miss.
- 8 centroids × 8 buckets fits comfortably in < 2 MB RAM.

### Negative
- Cold start: first 72 h produces no alarms. Operator must wait.
- A persistent attacker can drift the baseline by acting "normal-ish"
  for weeks — known limitation of all unsupervised baselines.

### Neutral
- `k=8` and bucketing scheme are pragmatic defaults; finer granularity
  improves recall at the cost of warm-up time.

## Alternatives considered

- **Cloud-trained anomaly model.** Rejected: privacy, offline gap, and
  the model would not be site-specific.
- **Rule-based schedule allowlist.** Rejected: brittle, requires
  per-site configuration the operator usually cannot provide.

## Plugin invocation

- `/behavioral-profiler` — install if needed, start, tail logs
- `/behavioral-profiler --once` — one-shot via `/console` with `--once`
- `/behavioral-profiler --console "<args>"` — arbitrary args
- `/behavioral-profiler --stop` — stop cog
- `/behavioral-profiler --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/behavioral-profiler/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `intrusion-detect-ml`, `panic-motion`.
