# ADR: Confined Space as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `confined-space`
**Category**: industrial

## Context

OSHA 1910.146 requires an attendant outside any permit-required confined
space (tank, vault, silo). Compliance is paper-driven and the attendant
frequently steps away. A worker collapsed inside the space may not be
discovered in time. A seed mounted at the entrance can independently
verify both that an attendant is present at the portal and that the
worker inside is still moving.

A useful confined-space monitor on a Pi Zero 2 W must:

1. Detect entry, presence, and exit at the portal.
2. Detect motion-stillness inside the space via a paired interior seed
   (mesh).
3. Escalate fast: 30 s of interior stillness while the worker has not
   exited is a man-down event.

## Decision

`confined-space` runs at 2 s cadence on `cog-sensor-sources`. State
machine: `vacant → attended-empty → occupied → stillness-warn →
MAN-DOWN`. The attendant zone (portal-side seed) and worker zone
(interior seed) are configured at install. Loss of attendant during an
occupied state fires `ATTENDANT-LOST`. Output: portal state, interior
state, occupancy duration, last interior motion delta.

As a Claude Code plugin, `/confined-space` returns the live state, the
incident log, and supports manual entry/exit acknowledgment.

## Consequences

### Positive
- Independent backstop on the OSHA-required human attendant.
- Paired-seed design is cheap and works in metal-walled spaces where
  GPS, BLE, and cell are useless.
- 2 s cadence catches a collapse well within survivable windows.

### Negative
- Requires two seeds per permit-space (portal + interior). Cost is
  still a fraction of a vendor confined-space stack.

### Neutral
- Stillness-warn threshold is tunable to the work being done (a welder
  may be motionless for legitimate seconds at a time).

## Alternatives considered

- **Worn gas/man-down beacon.** Complementary, not replacement —
  beacons fail if the worker is unconscious before activating, and
  battery management is a known compliance gap.
- **Single-seed install.** Rejected: cannot independently verify the
  attendant.

## Plugin invocation

- `/confined-space` — install if needed, start, tail logs
- `/confined-space --once` — current state snapshot
- `/confined-space --console "<args>"` — arbitrary args
- `/confined-space --stop` — stop
- `/confined-space --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/confined-space/`
- ADR-001 (foundational).
- `forklift-proximity`, `clean-room`.
