# ADR: Ghost Hunter as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `ghost-hunter`
**Category**: research

## Context

The novelty / paranormal-investigator audience wants a flashy "did
something just happen?" readout that the existing motion and audio cogs
won't surface, because those are tuned for high-precision, low-recall
events. Ghost-hunter is the inverse: high-recall, deliberately
sensitive, and entertaining. The honest description is that it is a
multi-channel anomaly detector that flags any frame whose joint
distribution of the 8 ESP32 features sits in the tail of recent
history. The "ghost" framing is product, not science.

Signal-processing approach: per-channel Welford running mean/variance
over a 10-minute window, then a Mahalanobis-style score across the
channel vector. A score above 3.5 sigma fires an "anomaly" event with a
plain-English breakdown of which channels diverged (e.g., "amplitude
quiet, variance up, dominant-bin shifted low"). No spirits required.

## Decision

Standalone armhf binary, reads `cog-sensor-sources` at 10 s cadence
default. Pipeline: rolling Welford → per-channel z-score → max-z and
sum-of-squared-z → threshold gate → JSON event with the contributing
channels listed. State machine is **calm → curious → anomaly →
cooldown**, where curious is a single-channel deviation and anomaly is
a multi-channel one.

As Claude Code plugin: `/ghost-hunter` wraps cog endpoints.

## Consequences

### Positive
- Fun front-end for a genuinely useful sensor: surfaces "something
  changed in the room" events the other cogs ignore.
- Self-calibrating — no thresholds to tune per-deployment.
- Output explains *why* it fired, which makes it debuggable.

### Negative
- High false-positive rate by design; not a security trigger.
- 10-minute baseline means it goes quiet during long stable periods,
  then over-fires when the room changes (HVAC kicks on, etc.).

### Neutral
- Sensitivity is a product choice, not a bug. Tune via `--interval`.

## Alternatives considered
- **Threshold per channel.** Rejected: misses joint anomalies that are
  unremarkable on any single channel.
- **One-class SVM.** Rejected: not worth the binary-size cost vs.
  Welford for the same effective AUC at this signal-to-noise.

## Plugin invocation
- `/ghost-hunter` install, start, tail logs
- `/ghost-hunter --once`
- `/ghost-hunter --console "--interval 5"`
- `/ghost-hunter --stop`
- `/ghost-hunter --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB (8-channel Welford state + small ring)
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/ghost-hunter/`
- ADR-001 foundational
- Related cogs: `coherence-gate`, `fall-detect` (also variance-driven)
