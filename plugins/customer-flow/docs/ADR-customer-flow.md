# ADR: Customer Flow as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `customer-flow`
**Category**: retail

## Context

Footfall is the most-used retail KPI and the most poorly measured.
Mechanical counters double-count groups, IR beams miss tailgaters, and
camera-based counters trip privacy regulations in EU and CA jurisdictions.
A small store wants accurate hourly in/out counts per door without
installing a camera or paying a SaaS subscription.

A useful flow counter on a Pi Zero 2 W must:

1. Distinguish ingress from egress (direction) at the threshold.
2. Resolve groups (a family of four) without identifying anyone.
3. Survive variable lighting and HVAC airflow that fool optical sensors.

## Decision

`customer-flow` watches the directional component of `cog-sensor-sources`
(CSI Doppler sign + amplitude rise/fall asymmetry) at 5 s intervals,
classifying each event as `in`, `out`, or `noise`. A two-second
debounce groups multi-person crossings. Hourly buckets persist to local
SQLite; the mesh aggregates per-door counts into a building total.
Output: `{in, out, net, hour, door_id, confidence}`.

As a Claude Code plugin, `/customer-flow` exposes today's running totals,
yesterday's reconciled buckets, and a 7-day comparison via the seed API.

## Consequences

### Positive
- No camera, no PII, GDPR-friendly: only anonymous count timestamps.
- Direction-aware — actual flow, not just presence ticks.
- Mesh federation gives multi-door stores a single number.

### Negative
- Tailgating still under-counts when two people cross within ~300 ms.
- Threshold geometry matters — a wide entrance may need two seeds.

### Neutral
- Calibration to door geometry is a one-time setup on install.

## Alternatives considered

- **Overhead stereo camera.** Rejected: cost, install, privacy.
- **Single IR beam.** Rejected: no direction, no group resolution.

## Plugin invocation

- `/customer-flow` — install if needed, start, tail logs
- `/customer-flow --once` — current hour snapshot
- `/customer-flow --console "<args>"` — arbitrary args
- `/customer-flow --stop` — stop
- `/customer-flow --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/customer-flow/`
- ADR-001 (foundational).
- `queue-length`, `dwell-heatmap`.
