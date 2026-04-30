# ADR: Gesture Recognition as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `gesture`
**Category**: developer

## Context

Many higher-level cogs (`gesture-language`, `music-conductor`, `dtw-gesture-learn`) need a stable, low-cost "is the person currently moving in a way that looks like a discrete gesture" primitive. Building that detection logic into each consumer cog duplicates Welford stats, debounce, and threshold tuning across the fleet.

A gesture, in this context, is a short bounded burst of motion — distinct from the slow background variance of normal occupancy and from the impulse spike of `fall-detect`. It must run on a Pi Zero 2 W with no camera and no wearable, using only the 8-feature ESP32 motion stream.

## Decision

`gesture` is a building-block cog, not a UX feature. It runs at 5 Hz over `cog-sensor-sources`, computes rolling variance + mean amplitude, and fires a `gesture_event` whenever motion energy crosses an upper band, sustains for ≥3 frames, and falls back into baseline within `max_gesture_duration_secs`. Output is a JSON envelope on stdout for downstream cogs to subscribe to via the seed event bus.

## Consequences

### Positive
- Consumer cogs share one tuned detector — no duplicated tuning per cog.
- 6 KB binary; trivial to ship in default seed image.
- 5 Hz duty cycle leaves headroom for 3–4 concurrent cogs.

### Negative
- Detects *that* a gesture happened, not *which* gesture. Classification is the consumer cog's job.
- Same 8-feature stream as a dozen other cogs — a fault upstream silences all of them.

### Neutral
- `interval` is exposed but most consumers run it at default 5 s; tuning is rarely needed.

## Alternatives considered

- **Inline detection in each consumer cog.** Rejected: tuning drift between cogs, no shared baseline.
- **Camera-based MediaPipe gesture.** Rejected: no camera on default seed; RAM budget forbids it.

## Plugin invocation
- `/gesture` install, start, tail
- `/gesture --once`
- `/gesture --console "--interval 5"`
- `/gesture --stop`
- `/gesture --logs`

## Resource budget
- Binary: ~410 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/gesture/` | ADR-001 | ADR-002 (fall-detect, similar state machine) | gesture-language
