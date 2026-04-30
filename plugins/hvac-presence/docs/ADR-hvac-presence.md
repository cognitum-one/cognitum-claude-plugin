# ADR: HVAC Presence as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `hvac-presence`
**Category**: building

## Context

Commercial HVAC waste is dominated by conditioning empty space.
Setback schedules (programmable thermostats) help but cannot react to
unscheduled arrivals or early departures. The seed already detects
human presence via ambient features; relaying that presence to the
HVAC controller closes the loop. Dedicated presence-controlled
thermostats exist (Ecobee, Nest) but tie the customer to a particular
stack and don't extend to multi-zone commercial split systems.

The signal-processing approach is straightforward: a **debounced
threshold over the motion-feature stream** with separate timeouts for
arrival (fast trigger, e.g. 30 s sustained motion) and departure (slow,
e.g. 10 min sustained quiet) avoids flapping the compressor.
Hysteresis is the entire signal-processing story — sophistication here
is a bug, not a feature.

## Decision

Standalone armhf binary on seed. Reads `cog-sensor-sources` every 10 s.
Computes a 30 s moving-average motion energy. State machine:
`unoccupied → arriving → OCCUPIED → leaving → unoccupied`. Transitions
emit JSON deltas which a downstream HVAC bridge cog (or external
relay) consumes as setpoint commands. Configurable arrival/departure
timeouts. Packaged as Claude Code plugin: slash command
`/hvac-presence` wraps seed's cog management endpoints.

## Consequences

### Positive
- Trivial logic, near-zero CPU, ideal for small Pi Zero 2 W class.
- Reuses the same ambient features that drive other building cogs —
  no extra sensor cost.
- Hysteresis prevents short-cycling, which protects HVAC equipment.

### Negative
- Single-zone view only. Multi-zone systems need one cog instance per
  zone (cheap on the seed but operator must wire them up).
- Sleeping occupants produce minimal motion — long unoccupied window
  may falsely trigger setback.

### Neutral
- 10 min departure timeout is a reasonable default; some buildings
  prefer 20 min to be conservative.

## Alternatives considered

- **Schedule-based setback.** Rejected as the *primary* control:
  doesn't react to unscheduled occupancy. Schedules remain a useful
  fallback when the cog is offline.
- **Per-room CO₂ sensor.** Rejected for v1: another sensor to install.
  Could augment in v2 for sleeping-occupant detection.

## Plugin invocation

- `/hvac-presence` — install if needed, start, tail logs
- `/hvac-presence --once` — one-shot via `/console` with `--once`
- `/hvac-presence --console "<args>"` — arbitrary args
- `/hvac-presence --stop` — stop cog
- `/hvac-presence --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/hvac-presence/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `occupancy-zones`, `energy-audit`.
