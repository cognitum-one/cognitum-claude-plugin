# ADR: Occupancy Zones as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `occupancy-zones`
**Category**: building

## Context

Building automation needs to know **how many people are in each room**
to size HVAC, schedule cleaning, audit fire-code load, and target
lighting. PIR motion sensors give a binary present/absent and miss
multiple occupants. Cameras work but are privacy-hostile and require
per-room hardware. WiFi CSI (Channel State Information), already
streaming from the ESP32 RuView companion, encodes per-subcarrier
amplitude perturbations that scale with the *number* of multipath
scatterers — i.e., people — and crucially **passes through walls**.

The signal-processing approach is **subcarrier-variance autocorrelation**:
per-zone CSI variance is summed across the human-doppler band (1-4 Hz),
then mapped to occupancy count via a piecewise-linear calibration curve
trained per zone during commissioning. Multiple seeds in mesh provide
spatial diversity, with the per-seed estimates fused via weighted average.

## Decision

Standalone armhf binary on seed. Reads ESP32 CSI features from
`cog-sensor-sources` every 5 s. Computes per-zone variance, applies
calibration map, emits zone-level counts. State machine: `idle →
sampling → fused`. Calibration is loaded from `/etc/cognitum/
occupancy-zones.toml`; commissioning mode walks the operator through
zone-by-zone counts. Emits JSON with `zone_id`, `count_estimate`,
`confidence`, `seed_count_in_fusion`. Packaged as Claude Code plugin:
slash command `/occupancy-zones` wraps seed's cog management endpoints.

## Consequences

### Positive
- Through-wall: one seed often covers 2-3 adjacent rooms.
- Privacy-preserving — no images, no MAC tracking required.
- Mesh fusion improves count accuracy beyond any single seed.

### Negative
- Requires per-zone calibration; uncalibrated rooms produce only
  "occupied yes/no", not counts.
- Confidence collapses with fewer than 2 seeds in mesh.

### Neutral
- ±1 person accuracy in calibrated zones is typical; finer counts need
  more seeds or explicit ground-truth retraining.

## Alternatives considered

- **PIR per zone.** Rejected: binary, costly to retrofit, misses
  multiple occupants.
- **Camera-based people counting.** Rejected: privacy + per-room
  hardware cost + RAM budget.

## Plugin invocation

- `/occupancy-zones` — install if needed, start, tail logs
- `/occupancy-zones --once` — one-shot via `/console` with `--once`
- `/occupancy-zones --console "<args>"` — arbitrary args
- `/occupancy-zones --stop` — stop cog
- `/occupancy-zones --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/occupancy-zones/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `hvac-presence`, `meeting-room`.
