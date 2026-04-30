# ADR: Sleep Apnea Detector as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `sleep-apnea`
**Category**: health

## Context

Obstructive sleep apnea affects an estimated 1 in 5 adults and is heavily
underdiagnosed because the gold-standard polysomnography exam requires an
overnight clinic stay. The clinical signal of interest is simple: a
breathing-rate envelope that **drops to zero (or near-zero) for ≥10 s**,
repeated over the night.

A bedroom seed at 169.254.42.1 already exposes a chest-motion / breathing
proxy in the shared `cog-sensor-sources` feature stream. Detecting apnea
events is well within reach of a small Pi Zero 2 W cog: bandpass-filter
the breathing band (~0.1–0.5 Hz), track the rolling envelope, and count
sustained dropouts. No camera, no worn device, no cloud upload.

This is a **screening / trending** tool, not a diagnostic — the value is
nightly event counts trended over weeks, which can prompt a real sleep
study.

## Decision

The cog runs as a standalone armhf binary on the seed, reading the shared
`cog-sensor-sources` feature stream every `interval` seconds (default 5s).
Each frame is fed through a breathing-band envelope detector; when the
envelope stays below a relative-to-baseline floor for ≥10 s the cog emits
an `APNEA_EVENT` JSON record and increments the night counter.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/sleep-apnea` that wraps the seed's cog management endpoints
(install, start, console, stop, logs).

## Consequences

### Positive
- Contactless, ambient — no CPAP, no chest strap, no nightly setup ritual.
- Per-night event count is the actually-useful clinical metric and falls
  out of the cog's state machine for free.
- Tiny binary (4 KB description footprint) leaves headroom for stacking
  `vital-trend` and `dream-stage` on the same seed for a fuller picture.

### Negative
- Cannot distinguish obstructive vs. central apnea without SpO2 — that
  requires a worn pulse-ox.
- Bed partner motion can mask one sleeper; the cog reports per-bed, not
  per-person, until a multi-seed mesh resolves zones.

### Neutral
- Event threshold (10 s) is the AASM-standard floor; tightening it
  trades sensitivity for false-positive rate per deployment.

## Alternatives considered

- **Send raw audio/IMU to cloud ML.** Rejected: bedroom audio is the
  most privacy-sensitive feed in the home; on-device thresholding avoids
  the upload entirely.
- **Require a worn pulse-ox.** Rejected for v1: defeats the "screening
  appliance" wedge. Could be a v2 mesh input.

## Plugin invocation

- `/sleep-apnea` — install if needed, start, tail logs
- `/sleep-apnea --once` — one-shot console execution with `--once`
- `/sleep-apnea --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--interval 3`)
- `/sleep-apnea --stop` — stop the running cog
- `/sleep-apnea --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/sleep-apnea/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related health cogs: `respiratory-distress` (acute version of the same
  signal), `dream-stage` (overnight context to interpret event clusters)
