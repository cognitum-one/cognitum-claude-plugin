# ADR: Forklift Proximity as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `forklift-proximity`
**Category**: industrial

## Context

Pedestrian-vs-forklift collisions are one of the top three OSHA
warehouse fatalities. Vendor proximity systems (RFID tags worn by
workers, ultrasonic on the lift) cost USD 3-5k per lift plus tag
provisioning, and many smaller 3PLs run without them. A passive seed
mounted at aisle intersections can detect the combined signature of a
moving lift (low-frequency rumble + characteristic CSI Doppler) and a
nearby pedestrian, and trigger an audible alert before either rounds the
corner.

A useful proximity detector on a Pi Zero 2 W must:

1. Distinguish forklift from foot traffic from a pallet jack.
2. Run at high cadence (1 s default) — collision windows are short.
3. Trigger local audio buzzer over GPIO; fire mesh broadcast to the
   neighbouring aisle's seed.

## Decision

`forklift-proximity` runs at 1 Hz on `cog-sensor-sources`, classifying
each frame into `clear`, `pedestrian`, `forklift`, or `both`. A
`both` state with a closing-velocity estimate above threshold transitions
to `WARN` and pulls the GPIO buzzer line. Mesh peers receive an alert
so the next aisle's beacons can pre-fire. Output: `state`, `closing_mps`,
`alert`, `last_clear_ts`.

As a Claude Code plugin, `/forklift-proximity` exposes the live state and
the day's WARN event log.

## Consequences

### Positive
- An order of magnitude cheaper than vendor proximity stacks.
- Mesh-aware: a chain of seeds gives blind-corner forewarning.
- Local audible alert does not depend on network — fail-safe at the seed.

### Negative
- Cannot identify which worker; only that a worker is present. Vendor
  RFID is still better for compliance audit trails.

### Neutral
- Closing-velocity threshold needs per-warehouse calibration based on
  speed limits and aisle geometry.

## Alternatives considered

- **RFID tag + lift-mounted reader.** Rejected on cost and tag
  provisioning friction; remains a complementary upgrade.
- **Lift-mounted LiDAR/ultrasonic.** Rejected: protects only the
  equipped lift, not the rest of the fleet.

## Plugin invocation

- `/forklift-proximity` — install if needed, start, tail logs
- `/forklift-proximity --once` — current state snapshot
- `/forklift-proximity --console "<args>"` — arbitrary args
- `/forklift-proximity --stop` — stop
- `/forklift-proximity --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/forklift-proximity/`
- ADR-001 (foundational).
- `confined-space`, `structural-vibration`.
