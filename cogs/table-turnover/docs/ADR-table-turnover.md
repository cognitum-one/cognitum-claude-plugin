# ADR: Table Turnover as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `table-turnover`
**Category**: retail

## Context

Restaurant table utilisation — seat time, idle gap between seatings,
turnover rate per shift — is usually inferred from POS timestamps, which
miss the gap between guest departure and busser cleanup. That gap is
where revenue leaks. A small ambient sensor that knows whether each
table is `occupied`, `seated-no-order`, `idle`, or `bussing` closes the
loop without requiring a camera over every booth.

A useful table monitor on a Pi Zero 2 W must:

1. Detect occupancy at the table-cluster level without identifying
   guests.
2. Distinguish "still eating" from "left, table not yet bussed".
3. Run at coarse 10 s intervals — turnover is minutes, not seconds.

## Decision

`table-turnover` runs at 10 s intervals on `cog-sensor-sources` features
spatially clustered to per-table zones (configured at install). A
four-state machine per table: `empty → occupied → idle (post-meal,
no motion) → bussing (transient activity, then empty)`. Transitions
out of `idle` without going through `bussing` flag a "ghost table".
Output: per-table state, dwell seconds, turns this shift, ghost count.

As a Claude Code plugin, `/table-turnover` returns the shift dashboard
and lets a manager drill into a specific table id.

## Consequences

### Positive
- Surfaces the post-meal idle gap that POS data hides.
- No camera over the diner, no PII, low manager-training burden.
- 10 s cadence keeps CPU well under budget on a busy floor.

### Negative
- Per-table zone calibration is required at install — moving tables
  invalidates zones until reconfigured.

### Neutral
- Ghost-table threshold (idle-without-bussing) is tunable per venue.

## Alternatives considered

- **Pressure-mat under each chair.** Rejected: install cost, durability,
  and false positives from bags.
- **POS-only inference.** Rejected: misses the bussing gap, which is
  the entire point.

## Plugin invocation

- `/table-turnover` — install if needed, start, tail logs
- `/table-turnover --once` — current shift snapshot
- `/table-turnover --console "<args>"` — arbitrary args
- `/table-turnover --stop` — stop
- `/table-turnover --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/table-turnover/`
- ADR-001 (foundational).
- `customer-flow`, `dwell-heatmap`.
