# ADR: Meeting Room as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `meeting-room`
**Category**: building

## Context

Calendar-booked meeting rooms suffer from two symmetric pathologies:
**ghost bookings** (booked but empty, blocking real users) and
**squatting** (occupied without booking, invisible to schedulers).
Knowing the *real* free/occupied state of a room — independent of
the calendar — closes both gaps. Cameras and badge readers are
privacy- and cost-prohibitive in casual meeting rooms. The seed,
already mounted in the building, can answer the question from
ambient features alone.

The signal-processing approach is a **two-band threshold with
hysteresis**: motion-energy band (continuous occupant motion) and
audio-energy band (voice activity) are each compared against
adaptive baselines. Either band sustained above its baseline for
60 s flags occupied; both below baseline for 5 min flags free. Two
bands together cut both false positives (HVAC vibration alone) and
false negatives (a quiet solo worker).

## Decision

Standalone armhf binary on seed. Reads `cog-sensor-sources` every
10 s. Per-room state machine: `free → checking → OCCUPIED → vacating
→ free`. Optional integration with calendar via a downstream
`calendar-bridge` cog reconciles real state with bookings and emits
ghost/squatter events. Outputs JSON with `state`, `motion_energy`,
`audio_energy`, `seconds_in_state`. Packaged as Claude Code plugin:
slash command `/meeting-room` wraps seed's cog management endpoints.

## Consequences

### Positive
- Catches both ghost bookings and squatters with one cog.
- No camera, no badge — privacy-preserving.
- 10 s cadence is plenty for a use case that updates on minute-scales.

### Negative
- Quiet solo worker can be misclassified during long pauses; 5 min
  vacate window mitigates but doesn't eliminate.
- Cannot identify *who* is in the room — that's a calendar concern.

### Neutral
- Default thresholds suit typical 4-12 person rooms; very large or
  acoustically dead rooms benefit from per-room calibration.

## Alternatives considered

- **Badge-tap occupancy.** Rejected: user friction, doesn't catch
  squatters who didn't tap.
- **Camera-based.** Rejected: privacy + RAM budget. Calendar UI
  doesn't justify a camera.

## Plugin invocation

- `/meeting-room` — install if needed, start, tail logs
- `/meeting-room --once` — one-shot via `/console` with `--once`
- `/meeting-room --console "<args>"` — arbitrary args
- `/meeting-room --stop` — stop cog
- `/meeting-room --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/meeting-room/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `occupancy-zones`, `hvac-presence`.
