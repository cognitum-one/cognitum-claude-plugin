# ADR: Shelf Engagement as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `shelf-engagement`
**Category**: retail

## Context

Brands pay for endcap and eye-level shelf placement on the assumption
that pickup translates into purchase, but actual touch-vs-buy ratios are
opaque without a planogram-aware camera. Independent retailers, in
particular, have no data on which SKUs get handled but put back. A
seed mounted at shelf-end can pick up the close-range CSI perturbation
of a hand reaching into a specific bay and flag it as engagement.

A useful shelf-engagement detector on a Pi Zero 2 W must:

1. Distinguish an arm-into-bay reach from a passer-by.
2. Bin engagements by bay (left/centre/right of fixture).
3. Pair engagements with subsequent put-back vs. cart-add (planogram +
   weight sensor optional).

## Decision

`shelf-engagement` watches the close-range high-frequency components of
`cog-sensor-sources` for the characteristic "approach → reach → retract"
signature. State machine: `idle → approach → reach (bay N) →
retract`. Each completed cycle increments engagement count for bay N.
A 5 s sampling cadence keeps CPU low. Output: per-bay engagement count,
per-bay reach duration, total engagements this hour.

As a Claude Code plugin, `/shelf-engagement` returns hourly engagement
buckets per bay and a 7-day comparison.

## Consequences

### Positive
- No camera, no SKU recognition needed — bay-level granularity is
  sufficient for merchandising decisions.
- Brands get actual engagement-per-SKU data their CPG vendors will pay for.

### Negative
- Cannot distinguish "picked up and bought" from "picked up and put
  back" without a paired weight or POS feed.

### Neutral
- Bay count and width are calibrated per fixture at install.

## Alternatives considered

- **Smart-shelf weight sensors.** Rejected on retrofit cost and
  per-SKU calibration burden.
- **Computer vision over the aisle.** Rejected: privacy and cost.

## Plugin invocation

- `/shelf-engagement` — install if needed, start, tail logs
- `/shelf-engagement --once` — current hour snapshot
- `/shelf-engagement --console "<args>"` — arbitrary args
- `/shelf-engagement --stop` — stop
- `/shelf-engagement --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/shelf-engagement/`
- ADR-001 (foundational).
- `dwell-heatmap`, `customer-flow`.
