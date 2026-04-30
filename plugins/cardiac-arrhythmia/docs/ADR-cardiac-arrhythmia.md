# ADR: Cardiac Arrhythmia as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `cardiac-arrhythmia`
**Category**: health

## Context

Atrial fibrillation alone is undiagnosed in roughly a third of the people
who have it, and it is a leading cause of stroke. The clinical detection
problem boils down to **R-R interval irregularity** — the time between
heartbeats varies in a non-physiological way. That signal lives in the
0.8–2.0 Hz band of the seed's chest-motion proxy and is recoverable
without electrodes.

Target deployment is residential and assisted-living: a bedroom or
livingroom seed continuously inspecting the cardiac band of the shared
`cog-sensor-sources` stream and flagging irregular runs. Like
`sleep-apnea` this is **screening**, not diagnosis — the goal is to put
the user in front of a 12-lead ECG sooner, not to replace one.

Approach: bandpass to the cardiac band, peak-pick beats, then compute
RMSSD and a Poincaré-style scatter on a rolling window. Sustained
irregularity beyond a z-score threshold fires the alert.

## Decision

The cog runs as a standalone armhf binary on the seed, reading
`cog-sensor-sources` every `interval` seconds (default 5s, configurable
1–3600s for low-duty trending). Beat detection runs on the 0.8–2.0 Hz
bandpassed signal; the irregularity score is windowed to suppress
single-beat noise. Persistent irregularity emits `ARRHYTHMIA_DETECTED`.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/cardiac-arrhythmia` that wraps the seed's cog management
endpoints (install, start, console, stop, logs).

## Consequences

### Positive
- Contactless monitoring on subjects who would never wear a Holter or
  a smartwatch (elderly, dementia, post-stroke recovery).
- Long observation windows — the seed runs 24/7, so paroxysmal afib is
  far more likely to be caught than in a 24-hour Holter snapshot.
- The 1–3600s `interval` range lets ops trade responsiveness vs. CPU
  for low-power deployments.

### Negative
- Cannot replace ECG: no waveform morphology, no ST analysis, no QT.
  It is strictly an irregularity flag.
- Motion artifacts (the user pacing, vacuuming) corrupt the cardiac
  band; the cog must gate on a stillness pre-condition or false-positive.

### Neutral
- Default 5s interval is conservative for a bedroom; a livingroom
  install will likely want 15–30s to amortize motion gating.

## Alternatives considered

- **Wearable PPG / ECG patch broadcast over BLE.** Rejected for v1:
  the wedge is "monitor people who refuse wearables." Can be added as
  a higher-confidence secondary input later.
- **Cloud ML on raw waveform.** Rejected: privacy and Pi Zero 2 W
  bandwidth budget. Local thresholding catches the high-yield cases.

## Plugin invocation

- `/cardiac-arrhythmia` — install if needed, start, tail logs
- `/cardiac-arrhythmia --once` — one-shot console execution with `--once`
- `/cardiac-arrhythmia --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--interval 30`)
- `/cardiac-arrhythmia --stop` — stop the running cog
- `/cardiac-arrhythmia --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/cardiac-arrhythmia/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related health cogs: `vital-trend` (provides the long-term HR baseline
  this cog z-scores against), `respiratory-distress` (shares the
  bandpass front-end machinery)
