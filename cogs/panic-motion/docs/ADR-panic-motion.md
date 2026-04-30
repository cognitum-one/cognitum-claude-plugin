# ADR: Panic Motion as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `panic-motion`
**Category**: security

## Context

Panicked or erratic motion — a struggle, a flee, a sudden scuffle — has a
characteristic frequency-domain signature: high jerk (3rd derivative of
position), broadband variance, and breakdown of the normal periodic gait
of a walking person. The seed already ships `gait-analysis` for slow
biomarker drift and `fall-detect` for the impact-then-stillness signature,
but neither catches the *agitated-but-still-upright* pattern that precedes
many security incidents (assault, abduction, medical distress).

Target deployment: retail back-of-house, lone-worker sites, eldercare
common rooms, parking structures. Cloud ML or dedicated PIR/radar panic
buttons exist but require either privacy-invasive video upload or
explicit user activation — the very thing a victim under duress cannot do.

## Decision

Standalone armhf binary on seed (Pi Zero 2 W class). Reads
`cog-sensor-sources` feature stream at 1 s sampling. Computes a rolling
**jerk z-score** (3rd-derivative variance vs. 60 s baseline) and a
**spectral flatness** ratio over the motion-band FFT. State machine:
`calm → agitated → PANIC` requires both metrics > threshold for 3
consecutive frames. Emits structured JSON with `panic_detected`,
`jerk_z`, `spectral_flatness`, `confidence`. Packaged as Claude Code
plugin: slash command `/panic-motion` wraps seed's cog management
endpoints.

## Consequences

### Positive
- No wearable, no button, no camera — works from ambient features alone.
- Distinguishes panic from exercise (exercise has *periodic* high jerk).
- Two-metric gate cuts false positives from kids running, vacuum cleaners.

### Negative
- Cannot identify *who* is panicking, only that someone is.
- Tuning thresholds varies by venue — gym deployment needs recalibration.

### Neutral
- Confidence score is heuristic; calibrated alarming requires fleet stats.

## Alternatives considered

- **Wearable panic button.** Rejected: requires victim action under duress.
- **Video pose estimation.** Rejected: privacy + Pi Zero 2 W RAM budget
  cannot host a real-time pose model.

## Plugin invocation

- `/panic-motion` — install if needed, start, tail logs
- `/panic-motion --once` — one-shot via `/console` with `--once`
- `/panic-motion --console "<args>"` — arbitrary args
- `/panic-motion --stop` — stop cog
- `/panic-motion --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/panic-motion/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `fall-detect`, `behavioral-profiler`.
