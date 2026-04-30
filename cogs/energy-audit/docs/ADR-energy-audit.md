# ADR: Energy Audit as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `energy-audit`
**Category**: building

## Context

Most commercial buildings waste 20-40% of their energy on
conditioning and lighting empty space outside of operational hours.
A schedule-based audit (compare meter to occupancy) usually requires
either a costly EMS install or an expensive consultant. The seed
already observes occupancy and ambient features and can correlate
those against a smart meter feed (or, in the absence of one, a
modeled load proxy).

The signal-processing approach is **autocorrelation over a 168-hour
(weekly) window** to learn the building's natural duty cycle, then
**residual analysis** — for each hour, compare actual occupancy-vs-
load against the learned weekly baseline. Persistent positive
residuals (load present, occupancy absent) flag the dollars being
wasted; persistent negative residuals (occupancy present, low load)
flag setback opportunities.

## Decision

Standalone armhf binary on seed. Default 60 s cadence; learns over a
rolling 4-week window. Inputs: occupancy from `occupancy-zones`,
load from a meter cog (or an estimated proxy if no meter). State
machine: `learning → baselined → reporting`. Outputs daily and
weekly waste estimates in kWh and currency, plus per-hour residual
heatmap data. Emits JSON with `hour_of_week`, `residual_kwh`,
`waste_dollars_estimate`, `confidence`. Packaged as Claude Code
plugin: slash command `/energy-audit` wraps seed's cog management
endpoints.

## Consequences

### Positive
- Self-baselining — works in any building without manual schedule
  authoring.
- Quantifies waste in dollars, not just anomaly scores; directly
  actionable for facilities owners.
- 60 s cadence is light; runs alongside higher-rate cogs without
  contention.

### Negative
- 4-week warm-up before useful output. Holiday/summer-shutdown
  periods skew the baseline transiently.
- Without a meter feed, the load-proxy estimate is approximate (±15%).

### Neutral
- Weekly autocorrelation is the right window for office buildings;
  retail and 24/7 facilities benefit from a longer window.

## Alternatives considered

- **Static schedule audit.** Rejected: requires hand-authored
  schedules that are usually wrong or stale.
- **Cloud BMS analytics.** Rejected for self-contained seed
  deployments; remains a possible upstream destination for the
  cog's emitted metrics.

## Plugin invocation

- `/energy-audit` — install if needed, start, tail logs
- `/energy-audit --once` — one-shot via `/console` with `--once`
- `/energy-audit --console "<args>"` — arbitrary args
- `/energy-audit --stop` — stop cog
- `/energy-audit --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/energy-audit/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `hvac-presence`, `occupancy-zones`.
