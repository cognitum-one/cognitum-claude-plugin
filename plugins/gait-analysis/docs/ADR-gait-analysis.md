# ADR: Gait Analysis as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `gait-analysis`
**Category**: health

## Context

Falls in older adults are predicted weeks-to-months in advance by gait
biomarkers — stride time variability, cadence asymmetry, and walking
speed decline. Clinical gait labs measure these with force plates or
worn IMUs, neither of which scales to "ambient monitoring of grandma."

Target deployment is residential elder-care and assisted-living
hallways. The signal of interest is the **periodic peak structure** of
the motion proxy in the 1–2.5 Hz band (one peak per heel-strike). From
that you can derive: cadence (BPM-equivalent), stride-time variance
(coefficient of variation), and a cadence asymmetry score (left/right
peak interval ratio).

Approach: peak-pick the bandpassed motion stream, accumulate inter-peak
intervals over a walking bout (10s+ of continuous activity), then
compute CV and asymmetry. Drift in any biomarker against the user's own
30-day baseline is the meaningful signal — absolute thresholds are
useless across body types.

## Decision

The cog runs as a standalone armhf binary on the seed, reading
`cog-sensor-sources` every `interval` seconds (default 10s; up to 3600s
for low-duty trending). It detects walking bouts, extracts per-bout
biomarkers, and emits a daily fall-risk score that is the z-score of
today's biomarkers against a rolling 30-day baseline.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/gait-analysis` that wraps the seed's cog management endpoints
(install, start, console, stop, logs).

## Consequences

### Positive
- Predictive, not reactive — surfaces fall risk **before** the
  fall-event detectors (`fall-detect`) ever need to fire.
- No wearable; works from ambient feature stream as the user walks
  past the seed in a hallway.
- 10s default interval is plenty; gait drifts on a scale of weeks, not
  seconds.

### Negative
- Requires sustained walking bouts; sedentary days produce no signal
  and no update to the baseline.
- A single seed sees a single walking corridor; whole-house coverage
  needs a mesh of seeds in choke-point hallways.

### Neutral
- The 30-day baseline window means the first month is a learning phase
  with no risk score; ops should communicate this to deployers.

## Alternatives considered

- **Worn IMU (Apple Watch, FitBit).** Rejected for v1: the population
  most at risk is the population most resistant to wearing devices.
  Can be a v2 input via mesh.
- **Camera-based skeleton tracking.** Rejected: privacy hostile in
  bedrooms and bathrooms (the highest-fall-rate rooms), and busts the
  Pi Zero 2 W resource budget.

## Plugin invocation

- `/gait-analysis` — install if needed, start, tail logs
- `/gait-analysis --once` — one-shot console execution with `--once`
- `/gait-analysis --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--interval 30`)
- `/gait-analysis --stop` — stop the running cog
- `/gait-analysis --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/gait-analysis/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related cogs: `fall-detect` (the event detector this trending cog is
  the leading indicator for), `vital-trend` (sibling biomarker-trending
  pattern for cardiopulmonary signals)
