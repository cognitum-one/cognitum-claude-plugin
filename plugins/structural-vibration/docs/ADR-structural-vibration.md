# ADR: Structural Vibration as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `structural-vibration`
**Category**: industrial

## Context

Predictive maintenance on rotating machinery (HVAC fans, pumps, motors)
and structural-health monitoring on bridges and old buildings depend on
RMS vibration level and its dominant frequency. Industrial vendor
sensors (PCB Piezotronics, SKF) cost USD 1-3k per channel plus a
gateway. A seed colocated near the bearing or anchor point can sample
the on-board IMU and compute RMS plus a coarse FFT cheap enough to leave
running indefinitely.

A useful vibration monitor on a Pi Zero 2 W must:

1. Sample IMU at 100+ Hz and compute RMS over a sliding window.
2. Detect both the slow drift of a degrading bearing and the sudden
   spike of an event (impact, seismic, anchor failure).
3. Escalate over `--threshold` RMS to a hard alert; below it, just log.

## Decision

`structural-vibration` samples the IMU at default 100 Hz, computes RMS
over a 1 s window every tick, and runs a 256-point FFT each cycle for
the dominant peak. State machine: `nominal → drift-warn (RMS slope
above slope-threshold) → SPIKE (single frame > threshold) →
SUSTAINED (3+ consecutive frames > threshold)`. SUSTAINED is the
hard alert; SPIKE alone is suppressed unless followed by SUSTAINED.
Output: `rms`, `peak_hz`, `state`, `slope`, `last_spike_ts`.

As a Claude Code plugin, `/structural-vibration` exposes live RMS and the
day's spike/sustained event log.

## Consequences

### Positive
- Two orders of magnitude cheaper than vendor PdM channels.
- Catches both bearing degradation (slow drift) and discrete events
  (impact, fall, seismic).
- 1 s window + 100 Hz sample is well inside the Pi Zero 2 W budget.

### Negative
- Dynamic range and noise floor are below industrial-grade piezo
  sensors — useful for trend detection, not absolute calibrated
  diagnostics.

### Neutral
- `threshold` is the most-tuned config; default 50.0 is a starting
  point for typical HVAC monitoring and must be reset for civil
  structures.

## Alternatives considered

- **PCB/SKF piezo + dedicated DAQ.** Complementary for high-stakes
  channels — seed provides cheap broad coverage to triage which channel
  deserves the piezo.
- **Smartphone-based wave logger.** Rejected: not deployable
  unattended, battery and OS lifecycle wrong shape.

## Plugin invocation

- `/structural-vibration` — install if needed, start, tail logs
- `/structural-vibration --once` — current RMS snapshot
- `/structural-vibration --console "<args>"` — arbitrary args
- `/structural-vibration --stop` — stop
- `/structural-vibration --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/structural-vibration/`
- ADR-001 (foundational).
- `forklift-proximity`, `livestock-monitor`.
