# ADR: Dream Stage as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `dream-stage`
**Category**: health

## Context

Sleep architecture (light / deep / REM) is the variable patients ask
about most after "did I sleep well?" — and it's the variable consumer
wearables most aggressively oversell. The clinically defensible
proxies are: **breathing-rate variance** (low in deep, high in REM),
**body-motion micro-events** (lowest in deep, modest twitches in REM),
and **cardiac variability** (elevated in REM). PSG uses EEG; an
ambient seed cannot, but the three motion/respiration proxies above
get within useful agreement of the gold standard for trend purposes.

Target deployment is residential sleep-quality monitoring. Unlike
`sleep-apnea` (event detector) this cog is a **stage classifier** that
runs slowly (default 60s evaluations, range 10–600s) because sleep
stages don't change faster than minutes.

Approach: per evaluation window, compute breathing variance and motion
micro-event rate against the night's running baseline, then apply a
small fixed-rule classifier (Wake / Light / Deep / REM). Emit the
current stage and the cumulative time-in-stage for the night.

## Decision

The cog runs as a standalone armhf binary on the seed, reading
`cog-sensor-sources` every `interval` seconds (default 60s). State
transitions follow the sleep-medicine canonical order with a minimum
dwell time per stage to prevent flapping. On wake-up the cog emits a
night summary and resets.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/dream-stage` that wraps the seed's cog management endpoints
(install, start, console, stop, logs).

## Consequences

### Positive
- Contactless, no wrist-band tan-line, no headband — works from the
  bedside seed the user already has running for `sleep-apnea`.
- 60s evaluation interval is gentle on the Pi Zero 2 W; this cog
  comfortably stacks with `sleep-apnea` and `vital-trend` overnight.
- Time-in-stage trends across weeks reveal real lifestyle effects
  (caffeine, alcohol, screen time) that single-night results can't.

### Negative
- Stage labels are heuristic, not EEG-truth — calling them "REM" is
  the right user-facing word but the right clinical word is
  "low-motion-high-respiration-variance bin."
- Bed-partner motion can mask the subject; reliable per-person
  staging needs a single-occupant bed or a multi-seed mesh.

### Neutral
- Default 60s interval can be shortened (10s minimum) for research
  use cases at the cost of more CPU; longer intervals (up to 600s)
  reduce power for battery-backed deployments.

## Alternatives considered

- **Wrist-worn PPG-based staging (Apple Watch, Oura).** Rejected for
  v1: the wedge is "no wearables." Can be a complementary v2 input.
- **EEG headband (Muse, Dreem).** Rejected: low compliance — users
  wear them for a week, then never again. The seed is already plugged
  in and on the nightstand.

## Plugin invocation

- `/dream-stage` — install if needed, start, tail logs
- `/dream-stage --once` — one-shot console execution with `--once`
- `/dream-stage --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--interval 30`)
- `/dream-stage --stop` — stop the running cog
- `/dream-stage --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/dream-stage/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related cogs: `sleep-apnea` (event detector that pairs with this
  stage classifier overnight), `vital-trend` (provides the BR / HR
  baselines this classifier z-scores against)
