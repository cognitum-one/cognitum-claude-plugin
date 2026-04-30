# ADR: Elevator Count as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `elevator-count`
**Category**: building

## Context

Elevator overload trips (commonly 13-21 occupants depending on car
class) cause door-reset cycles, downtime, and code-violation incidents.
Existing solutions are weight-platform sensors (require service-tech
installation, expensive) or vision systems (privacy and lighting
issues — elevators are often dark and mirrored). A seed mounted on
the cab roof or above the door can use ambient WiFi CSI features
plus a barometric or motion-derived car-position cue to count people
in a fully enclosed metal box, where camera-based methods struggle.

The signal-processing approach combines two streams: (1)
**CSI subcarrier-variance summation** at floor-stop dwell windows
(when the car is stationary, multipath stabilizes and per-occupant
contribution is most measurable); (2) **doppler accumulation** during
boarding (autocorrelation peaks each time a person crosses the door
threshold). The two estimates are fused per stop.

## Decision

Standalone armhf binary on seed. Reads `cog-sensor-sources` every 5 s.
A car-state detector (motion plus barometric delta) gates measurement:
counting only happens during a stable dwell, with a fresh count after
each door cycle. Emits JSON per dwell event with `floor_estimate`,
`count`, `confidence`, `dwell_duration_ms`. State machine:
`moving → dwelling → counting → boarded`. Packaged as Claude Code
plugin: slash command `/elevator-count` wraps seed's cog management
endpoints.

## Consequences

### Positive
- Privacy-preserving (no camera) and works in dark/mirrored cabs.
- Single-seed solution — no service-tech weight-platform install.
- Mesh-friendly: lobby seeds can confirm boarding deltas with cab seed.

### Negative
- Calibration per cab car geometry is required; CSI signature differs
  by car size and metal lining.
- Counts are ±1 typical; not suitable for life-safety overload
  enforcement on its own.

### Neutral
- Door-cycle gating means counts update at floor stops, not
  continuously — fine for HVAC and operations, less so for live UI.

## Alternatives considered

- **Weight platform.** Rejected as primary: invasive install, high
  cost. Could remain as a hard-safety backstop.
- **Cab camera + people counting.** Rejected: privacy, low-light
  failure, RAM budget.

## Plugin invocation

- `/elevator-count` — install if needed, start, tail logs
- `/elevator-count --once` — one-shot via `/console` with `--once`
- `/elevator-count --console "<args>"` — arbitrary args
- `/elevator-count --stop` — stop cog
- `/elevator-count --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/elevator-count/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `occupancy-zones`, `meeting-room`.
