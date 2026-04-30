# ADR: Perimeter Breach as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `perimeter-breach`
**Category**: security

## Context

Single-zone intrusion detection answers "is something happening in the
room?" Multi-zone perimeter detection answers "**where** in the
property is something happening, and **which direction** is it moving?"
That second question is the one a guard or a homeowner actually wants
answered when an alarm fires.

Target deployment is small-commercial properties (warehouses, retail
back-of-house) and larger residential lots where multiple seeds can
each watch a zone (front door / side gate / back yard). The cog
consumes a multi-zone feature stream and tracks per-zone z-scores
plus the cross-zone activation order to estimate breach direction.

Approach: per zone, maintain a baseline and z-score current activity
(threshold default 3.0σ). When two adjacent zones cross threshold
within a short window, infer direction from the time order. Emit a
structured event with the zone list, direction estimate, and per-zone
confidence.

## Decision

The cog runs as a standalone armhf binary on the seed, reading the
multi-zone `cog-sensor-sources` stream every `interval` seconds
(default 2s — fast because direction inference needs short
inter-zone deltas). It owns one z-scorer per zone and a small
direction-classifier that consumes their event timestamps.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/perimeter-breach` that wraps the seed's cog management
endpoints (install, start, console, stop, logs).

## Consequences

### Positive
- Direction estimate ("entering vs. departing") changes the
  appropriate response — entering = call a guard, departing = call
  the police about what they took.
- Multi-zone resolution means a guard can be dispatched to the right
  side of the property instead of "somewhere on site."
- Per-zone baselines independently track each zone's normal activity
  (the loading-dock zone is genuinely louder than the back fence).

### Negative
- Requires multi-zone sensor coverage; deploying with a single zone
  collapses the cog to a worse `intrusion`.
- Direction inference assumes adjacent zones are physically adjacent
  in the deployment; misconfigured zone topology produces nonsense
  direction labels.

### Neutral
- 2s default scan interval is the floor for direction inference; a
  slower interval saves CPU but loses the "which way" signal.

## Alternatives considered

- **Single-zone repeated.** Rejected: gives up the direction signal,
  which is the main reason a perimeter cog exists.
- **Camera-based pose tracking.** Rejected for v1: privacy concerns
  in residential and the Pi Zero 2 W resource budget. Mesh of
  ambient seeds gets most of the yield.

## Plugin invocation

- `/perimeter-breach` — install if needed, start, tail logs
- `/perimeter-breach --once` — one-shot console execution with `--once`
- `/perimeter-breach --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--threshold 2.5`, `--interval 1`)
- `/perimeter-breach --stop` — stop the running cog
- `/perimeter-breach --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/perimeter-breach/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related security cogs: `intrusion` (single-zone parent of this
  multi-zone cog), `tailgating` (companion direction-aware cog at
  controlled-access points)
