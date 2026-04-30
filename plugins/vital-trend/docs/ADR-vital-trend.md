# ADR: Vital Trend as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `vital-trend`
**Category**: health

## Context

Single vital-sign measurements are noisy; **trends over weeks** are
clinically actionable. Resting heart rate creeping up 5 BPM over a
month is one of the strongest leading indicators of an impending
cardiac event, infection, or medication problem — but only if you have
the baseline.

Target deployment is residential continuous-care and post-discharge
monitoring. Unlike `respiratory-distress` (acute alerting) or
`cardiac-arrhythmia` (rhythm disorders), this cog is the
**biomarker-trending substrate** that the alerting cogs z-score
against. Without a long baseline, every other health cog is guessing.

Approach: configurable bandpass filters extract the breathing band
(default 0.1–0.5 Hz) and the cardiac band (default 0.8–2.0 Hz). The
cog computes per-window estimates of breathing rate (BPM) and heart
rate (BPM), persists them as a daily/weekly trend, and emits acute
alerts when either crosses a configured threshold (`tachypnea_threshold`
default 30 BPM, `tachycardia_threshold` default 100 BPM).

## Decision

The cog runs as a standalone armhf binary on the seed, reading
`cog-sensor-sources` every `interval` seconds (default 10s, range
1–3600s). It maintains rolling per-day median and IQR for both rate
metrics, exposes the trend on its console, and fires `TACHYPNEA` /
`TACHYCARDIA` events on threshold crossings. The acute thresholds and
the bandpass cutoffs are individually configurable so deployment to
adult vs. pediatric vs. athletic populations is just a config change.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/vital-trend` that wraps the seed's cog management endpoints
(install, start, console, stop, logs).

## Consequences

### Positive
- Provides the personal baseline that other health cogs need —
  `cardiac-arrhythmia` and `respiratory-distress` both consume the
  trend rather than each rebuilding their own.
- Configurable bandpass means one binary covers infants (faster
  breathing, faster HR) through adults without code changes.
- Cheap to run at long intervals (10s default) since the signal of
  interest is per-day, not per-second.

### Negative
- The first ~7 days of a fresh deployment have no useful trend; ops
  must communicate this baseline-learning period.
- Cannot disambiguate "user's HR is up because they're sick" from
  "user's HR is up because they took the stairs" without an activity
  signal — pair with `gait-analysis` for context.

### Neutral
- Default thresholds are conservative adult values; pediatric or
  athlete deployments will tune both the cutoff frequencies and the
  alert thresholds.

## Alternatives considered

- **Cloud trend analytics.** Rejected: trend data is the most
  identifying medical record there is; keeping it on-device is the
  whole point of the seed.
- **Hard-coded bandpass cutoffs.** Rejected: the user-tuned advanced
  config is what makes the same binary useful for a 6-month-old and
  a 75-year-old.

## Plugin invocation

- `/vital-trend` — install if needed, start, tail logs
- `/vital-trend --once` — one-shot console execution with `--once`
- `/vital-trend --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--interval 60`, `--breathing-low 0.15`)
- `/vital-trend --stop` — stop the running cog
- `/vital-trend --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/vital-trend/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related cogs: `cardiac-arrhythmia` (consumes this cog's HR baseline),
  `respiratory-distress` (consumes this cog's BR baseline)
