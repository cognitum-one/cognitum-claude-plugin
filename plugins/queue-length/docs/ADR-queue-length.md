# ADR: Queue Length as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `queue-length`
**Category**: retail

## Context

Checkout abandonment correlates strongly with perceived wait time: every
person past three in line measurably reduces conversion. Independent
retailers and quick-service restaurants need a queue signal but cannot
justify the cost of overhead camera analytics or LiDAR people-counters,
both of which also raise privacy concerns at the register.

A useful queue estimator on a Pi Zero 2 W must:

1. Distinguish a stationary queue from passers-by and staff motion.
2. Run cheap enough that one seed can also host neighbouring retail cogs
   (`dwell-heatmap`, `customer-flow`).
3. Avoid identifying individuals — only count and time them.

## Decision

`queue-length` is a standalone armhf binary that subscribes to the shared
`cog-sensor-sources` feature stream (CSI variance + microphone amplitude
envelope). A clustering pass groups dwell points into a one-dimensional
queue along the register axis, and a Little's-Law estimator (arrivals /
service rate) projects expected wait time. State machine: `empty →
forming → steady → surge → draining`. Sampling defaults to 5 s; surge
state escalates to 1 s. Output JSON includes `queue_len`, `eta_secs`,
`state`, `confidence`, `timestamp`.

As a Claude Code plugin, `/queue-length` wraps the seed's
`/api/v1/cogs/queue-length` endpoints — install if missing, start, tail
logs.

## Consequences

### Positive
- No camera, no PII — only anonymised RF/audio features.
- Surge alerts let staff open a second register before customers walk.
- Cheap enough to colocate with other retail cogs on one seed.

### Negative
- Single-seed estimate is line-of-the-register only; multi-aisle stores
  need one seed per checkpoint, federated via mesh.

### Neutral
- Wait-time confidence depends on a per-store calibration of service rate.

## Alternatives considered

- **Overhead camera + CV.** Rejected: privacy, cost, install complexity.
- **Beam-break counter at queue entry.** Rejected: counts entries but
  cannot estimate length once the queue snakes.

## Plugin invocation

- `/queue-length` — install if needed, start, tail logs
- `/queue-length --once` — one-shot
- `/queue-length --console "<args>"` — arbitrary args
- `/queue-length --stop` — stop
- `/queue-length --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/queue-length/`
- ADR-001 (foundational).
- `dwell-heatmap`, `customer-flow`.
