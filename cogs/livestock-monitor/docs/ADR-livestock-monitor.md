# ADR: Livestock Monitor as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `livestock-monitor`
**Category**: industrial

## Context

Small and mid-size farms (dairy parlours, swine barns, poultry sheds)
typically discover distress, illness, or escape through a once-a-shift
walkthrough. A sick calf at 02:00 is found at 06:00. Vendor barn-camera
analytics exist but cost USD 10k+ per barn and require steady internet.
A handful of seeds in a barn can pick up the diurnal vocalisation pattern
and motion baseline of a healthy herd, and flag deviations — the
unusual-quiet of a downed animal, the unusual-noisy of a panicked
group, or the unusual-empty of an open gate.

A useful livestock monitor on a Pi Zero 2 W must:

1. Learn a rolling per-barn baseline of activity and vocalisation by
   hour-of-day.
2. Detect three deviations: distress (high), abnormal-quiet, escape
   (zone went empty unexpectedly).
3. Survive dust, humidity, and ammonia in the barn environment without
   wiping the model on reboot.

## Decision

`livestock-monitor` accumulates `cog-sensor-sources` features into a
24-bin hour-of-day baseline (Welford running mean/variance, persisted
every 5 minutes to flash). At 5 s cadence it computes a z-score against
the current bin and emits one of `normal`, `distress`, `quiet-anomaly`,
`escape-anomaly`. A 30 s rolling window guards against single-frame
noise. Output: `state`, `z_activity`, `z_vocal`, `expected_band`,
`since_baseline_secs`.

As a Claude Code plugin, `/livestock-monitor` returns the live herd
status and the day's anomaly log.

## Consequences

### Positive
- Cheap enough to put one seed per pen rather than one camera per barn.
- Hour-of-day baseline adapts to feeding schedules and lighting cycles
  automatically — no manual tuning per season.
- Catches both "something is happening" and "something stopped
  happening" — the silence anomaly is what veterinary dashboards miss.

### Negative
- The baseline takes 48-72 hours to stabilise after first install or
  herd change.

### Neutral
- Z-score thresholds are tunable per species (poultry vs. cattle have
  very different vocalisation envelopes).

## Alternatives considered

- **Wearable bolus / ear-tag sensors.** Complementary — give per-animal
  data but at high per-head cost. Seed gives herd-level coverage cheaply.
- **Barn camera + cloud CV.** Rejected on cost, bandwidth, and
  intermittent rural connectivity.

## Plugin invocation

- `/livestock-monitor` — install if needed, start, tail logs
- `/livestock-monitor --once` — current herd snapshot
- `/livestock-monitor --console "<args>"` — arbitrary args
- `/livestock-monitor --stop` — stop
- `/livestock-monitor --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/livestock-monitor/`
- ADR-001 (foundational).
- `confined-space`, `structural-vibration`.
