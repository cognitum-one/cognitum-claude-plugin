# ADR: Coherence Gate as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `coherence-gate`
**Category**: signal

## Context

Several downstream cogs (sound-classifier, fall-detect, person-matching)
suffer when the input frame is dominated by noise — HVAC roar, fan
vibration, or simply the channel-cross-talk noise floor of the ESP32
itself. A shared upstream gate that suppresses incoherent frames is
cheaper than every consumer implementing its own denoise. The honest
description: this is a magnitude-squared coherence test between
adjacent frames, with hysteresis, dressed up under a more evocative
name.

Signal-processing approach: maintain the previous frame and an EMA of
the cross-frame correlation. The instantaneous "coherence" is the
absolute Pearson correlation between the current and previous
8-channel vectors, smoothed mildly. When coherence exceeds the
configured `--threshold` (default 0.7), the frame is forwarded; when
it falls below (with hysteresis to avoid flapping), the frame is
dropped and replaced in the output stream by a "noise" marker.

## Decision

Standalone armhf binary, reads `cog-sensor-sources`, publishes a gated
parallel stream. Pipeline: previous-frame buffer → Pearson correlation
→ EMA smoother → threshold gate with hysteresis (`th_high = threshold`,
`th_low = threshold - 0.1`). JSON sidecar reports `coherence`,
`is_open` (gate state), `frames_passed`, `frames_dropped`, and a
rolling pass-rate for observability.

As Claude Code plugin: `/coherence-gate` wraps cog endpoints.

## Consequences

### Positive
- One cheap gate cleans the input for all downstream cogs that
  subscribe through it.
- Hysteresis prevents on-off flapping at the threshold boundary.
- Pass-rate metric makes it obvious when the environment has gone
  hostile.

### Negative
- Falsely drops legitimate but uncorrelated transients (a single
  sharp clap looks "incoherent" by this metric).
- Threshold has to be re-tuned in unusually noisy environments.

### Neutral
- Default 0.7 is conservative; many deployments will lower it.

## Alternatives considered
- **Spectral coherence (cross-PSD).** Rejected: too expensive
  per-frame on Pi Zero 2 W and the time-domain Pearson is sufficient
  here.
- **Variance threshold alone.** Rejected: doesn't capture
  inter-channel structure.

## Plugin invocation
- `/coherence-gate` install, start, tail logs
- `/coherence-gate --once`
- `/coherence-gate --console "--threshold 0.6"`
- `/coherence-gate --stop`
- `/coherence-gate --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/coherence-gate/`
- ADR-001 foundational
- Related cogs: `flash-attention`, `sparse-recovery`
