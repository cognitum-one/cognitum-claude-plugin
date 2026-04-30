# ADR: Lighting Zones as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `lighting-zones`
**Category**: building

## Context

Standalone PIR-based lighting controls are common but flawed: they
miss small movements (sustained desk work), don't anticipate
inter-room transitions (corridor goes dark right when you walk in),
and don't share state across rooms. A connected lighting layer that
reads the same ambient feature stream the rest of the seed uses can
hand off illumination room-to-room as a person walks through the
space.

The signal-processing approach is **per-zone threshold-with-handoff**:
each zone computes a 5 s motion-energy average, but transitions
between zones bias neighbour zones to *anticipate* arrival —
specifically, when zone A is "vacating" (decreasing energy) and a
neighbouring zone B detected an arrival edge in the past 3 s, zone B
extends its on-window. This converts hard PIR cliffs into smooth
walking-through-a-house transitions.

## Decision

Standalone armhf binary on seed. Reads `cog-sensor-sources` every 5 s
per zone. Per-zone state machine: `dark → fading_up → ON → fading_down
→ dark`. Zone topology (which zones are adjacent) loaded from
`/etc/cognitum/lighting-zones.toml`. Transitions emit JSON commands
which a downstream lighting bridge cog (Zigbee2MQTT, Hue, KNX) routes
to fixtures. Packaged as Claude Code plugin: slash command
`/lighting-zones` wraps seed's cog management endpoints.

## Consequences

### Positive
- Smooth corridor-to-room handoffs eliminate the "dark for a second"
  lighting-cliff problem.
- Reads existing ambient features — no per-fixture occupancy sensor
  retrofit.
- Mesh-aware: a seed in zone A can prime zone B's seed via mesh msg.

### Negative
- Adjacency topology must be authored once. Auto-discovery is possible
  but not v1.
- Sleeping or motionless occupants will eventually trigger fade-down
  — same limit as any motion-based control.

### Neutral
- 5 s sampling is the smoothness/responsiveness sweet spot; faster
  is unnecessary for lighting.

## Alternatives considered

- **Per-fixture PIR.** Rejected: poor handoff, and standalone PIRs
  have no shared state.
- **Camera-based pose tracking.** Rejected: privacy + Pi Zero 2 W
  RAM budget.

## Plugin invocation

- `/lighting-zones` — install if needed, start, tail logs
- `/lighting-zones --once` — one-shot via `/console` with `--once`
- `/lighting-zones --console "<args>"` — arbitrary args
- `/lighting-zones --stop` — stop cog
- `/lighting-zones --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/lighting-zones/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `occupancy-zones`, `hvac-presence`.
