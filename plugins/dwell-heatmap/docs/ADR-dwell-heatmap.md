# ADR: Dwell Heatmap as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `dwell-heatmap`
**Category**: retail

## Context

Store layout decisions — endcap placement, fixture orientation, signage
visibility — are usually made on intuition and refreshed once a quarter
because the data needed to do better (cumulative dwell time per zone) is
expensive. Camera-based heatmaps require infrastructure most independent
retailers will not deploy. A passive seed-based heatmap that needs no
ceiling install and emits no images solves the same merchandising
question without the camera-shaped objections.

A useful dwell heatmap on a Pi Zero 2 W must:

1. Bin sensor activity into a low-resolution spatial grid without
   tracking individuals.
2. Smooth across hours/days so noise does not dominate the picture.
3. Run alongside `queue-length` and `customer-flow` on the same seed.

## Decision

`dwell-heatmap` accumulates `cog-sensor-sources` features into a coarse
grid (default 4×4 zones per seed footprint) and exponentially decays each
cell at a configurable half-life. Each tick (default 5 s) updates the
grid and emits the top-3 hottest zones plus a normalised matrix. Mesh
peers can federate grids into a store-level map.

As a Claude Code plugin, `/dwell-heatmap` wraps the seed's heatmap
endpoint, returning a compact JSON matrix that the harness can render in
the terminal.

## Consequences

### Positive
- No PII; only zone-level anomalous occupancy.
- Decay-weighted grid surfaces today's pattern, not last week's.
- Federates cleanly across a mesh of seeds for whole-floor maps.

### Negative
- Spatial resolution is intentionally coarse — not suitable for
  fixture-level A/B without dense seed deployment.

### Neutral
- Half-life choice trades responsiveness for stability; 1-hour default
  fits typical retail browsing.

## Alternatives considered

- **WiFi probe-request sniffing.** Rejected: MAC randomisation has
  largely killed the signal and raises privacy review concerns.
- **BLE beacon trilateration.** Rejected: requires customer cooperation
  via app install.

## Plugin invocation

- `/dwell-heatmap` — install if needed, start, tail logs
- `/dwell-heatmap --once` — one-shot snapshot
- `/dwell-heatmap --console "<args>"` — arbitrary args
- `/dwell-heatmap --stop` — stop
- `/dwell-heatmap --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/dwell-heatmap/`
- ADR-001 (foundational).
- `queue-length`, `shelf-engagement`.
