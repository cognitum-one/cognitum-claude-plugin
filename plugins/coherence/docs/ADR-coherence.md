# ADR: Coherence Monitor as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `coherence`
**Category**: developer

## Context

Most seed cogs read the same 8-feature ESP32 motion stream. When that stream degrades — a flaky USB cable, a saturating microphone, an ESP32 that just rebooted — every consuming cog quietly produces garbage. There is no shared "is my input trustworthy right now" signal, so each cog reinvents its own quality check, badly.

`coherence` watches the stream itself, not the phenomena inside it: are channels moving together when they should, is variance within historical bounds, are samples arriving on schedule.

## Decision

`coherence` runs at 0.1 Hz (default 10 s interval) over `cog-sensor-sources` and emits a per-channel score in `[0, 1]` plus an aggregate. The score combines: (a) inter-channel correlation versus a 5-minute rolling baseline, (b) variance ratio against the same baseline, (c) sample-arrival jitter. Below `0.5` aggregate, the cog publishes `signal_degraded` so downstream cogs can suppress alerts.

## Consequences

### Positive
- One source of truth for "trust the feature stream right now."
- 4 KB binary, near-zero CPU.
- Catches silent ESP32 brownouts that look like "everyone is asleep."

### Negative
- Adds a dependency edge: cogs that subscribe to `signal_degraded` go quiet when coherence itself is broken.

### Neutral
- "Coherence" here is the engineering sense (signal stability), not a dynamical-systems claim.

## Alternatives considered

- **Per-cog quality checks.** Rejected: drifts, duplicates effort, hides systemic faults.
- **Hardware watchdog only.** Rejected: catches dead ESP32 but not noisy/saturated ones.

## Plugin invocation
- `/coherence` install, start, tail
- `/coherence --once`
- `/coherence --console "--interval 10"`
- `/coherence --stop`
- `/coherence --logs`

## Resource budget
- Binary: ~400 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/coherence/` | ADR-001 | adversarial (related signal-integrity)
