# ADR: Plant Growth as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `plant-growth`
**Category**: research

## Context

Hobby growers, classroom science kits, and small indoor farms want a
"is the tomato actually growing?" readout without buying a dedicated
phytometer. The seed already pulls ESP32-side ambient features
(amplitude, variance, dominant-bin energy, and a coarse luminance proxy
derived from CSI-RSSI drift). What it lacks is a slow-rate aggregator
that turns those frames into a plant-relevant trend.

The signal-processing approach is intentionally simple: at the 5-minute
default cadence, plant changes are smaller than per-frame noise, so the
right tool is long-window low-pass filtering and circadian phase
detection — not a CNN. We exponentially smooth the luminance proxy to
distinguish day from night, and accumulate a "growth proxy" from the
day-segment variance of the spatial features (stable lighting + slowly
changing geometry = the ESP32 frame's mean shifts measurably across
weeks).

## Decision

Standalone armhf binary on the seed. Reads `cog-sensor-sources` at the
configured `--interval` (default 300 s). State machine: **dark →
twilight → light → twilight → dark**, driven by a 24 h sliding window
on the luminance proxy. Within each light segment the cog updates an
EMA of the spatial-feature centroid; the day-over-day delta in that
centroid is the reported growth proxy. Output is a single JSON line per
sample with `phase`, `lux_proxy`, `growth_index`, `daily_delta`, and
`days_observed`.

As Claude Code plugin: `/plant-growth` wraps cog endpoints — install,
start, tail, one-shot read, console-mode tuning.

## Consequences

### Positive
- Long-cadence default (300 s) keeps CPU idle most of the time.
- No camera required — circadian phase is recovered from the 8-feature
  stream alone.
- Daily delta is comparable across seeds in the same mesh.

### Negative
- Growth proxy is a relative trend, not centimetres; calibration to
  physical height needs an external reference once.
- Cannot distinguish leaf growth from the plant being moved.

### Neutral
- Works best in a stable indoor environment; outdoor placement adds
  weather-driven variance the cog cannot subtract.

## Alternatives considered
- **Camera + segmentation.** Rejected for v1: blows the RAM budget and
  needs a lens the seed doesn't ship with.
- **Soil-moisture probe input.** Rejected: requires extra hardware and
  doesn't measure growth, only stress.

## Plugin invocation
- `/plant-growth` install, start, tail logs
- `/plant-growth --once`
- `/plant-growth --console "--interval 600"`
- `/plant-growth --stop`
- `/plant-growth --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB (24 h ring buffer of 1-sample-per-5-min entries)
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/plant-growth/`
- ADR-001 foundational
- Related cogs: `happiness-score` (also EMA-based), `time-crystal`
