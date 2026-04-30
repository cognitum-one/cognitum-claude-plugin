# ADR: Weapon Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `weapon-detect`
**Category**: security

## Context

Concealed metal carries a distinctive RF / magnetometric signature
when it passes near a tuned antenna or magnetometer, well-explored in
the airport-security and event-security literature. A seed equipped
with the appropriate sensor add-on (or sourcing data from an upstream
RF feature stream) can perform the same kind of presence detection at
a doorway or vestibule. This is **not** a metal detector replacement
— it's a probabilistic flag that asks "should the human guard look
twice?"

Target deployment is commercial / institutional access control:
school entrances, courthouse vestibules, hospital ERs, event venues.
Residential is explicitly out of scope.

Approach: the upstream sensor exposes a metallic-signature feature in
the `cog-sensor-sources` stream. The cog z-scores that channel against
the empty-doorway baseline, applies a fast (default 1s) sampling
interval to catch a single person walking through, and emits
`METAL_SIGNATURE_DETECTED` with a confidence score on threshold
crossings.

## Decision

The cog runs as a standalone armhf binary on the seed, reading the
metallic-signature channel of `cog-sensor-sources` every `interval`
seconds (default 1s, range 1–60s). It maintains a long baseline
(empty doorway) and emits a JSON detection record with confidence
when a transit signal exceeds threshold. It does **not** lock doors,
sound alarms, or take any autonomous action — it informs a human
operator.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/weapon-detect` that wraps the seed's cog management
endpoints (install, start, console, stop, logs).

## Consequences

### Positive
- Adds a no-touch screening signal at controlled entry points without
  the cost or staffing of a walk-through detector.
- 1s default interval is fast enough to catch a person walking
  through at normal pace.
- Strict "advisory only" output makes the deployment legally and
  operationally safer than an autonomous-action design.

### Negative
- Cannot distinguish weapon from belt buckle, laptop, or large
  keyring on its own; it is a screening flag, not a classifier. False
  positives are expected and the deployment must staff for them.
- Requires a sensor add-on or upstream RF feature stream that not
  every seed carries; on a bare-microphone seed the cog has no
  signal and refuses to start.

### Neutral
- Threshold tuning per deployment is mandatory and ongoing — what
  reads as a signature in a marble lobby reads differently in a
  drywall vestibule.

## Alternatives considered

- **Walk-through metal detector.** Rejected as a replacement; the
  cog complements one rather than replacing it. Where a walk-through
  is too costly or slow, the cog is the next-best layer.
- **Camera + ML weapon detection.** Rejected for v1: false-positive
  rate on visually concealed weapons is poor, and the privacy and
  compute costs are large. Could be a v2 mesh input.

## Plugin invocation

- `/weapon-detect` — install if needed, start, tail logs
- `/weapon-detect --once` — one-shot console execution with `--once`
- `/weapon-detect --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--interval 2`)
- `/weapon-detect --stop` — stop the running cog
- `/weapon-detect --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/weapon-detect/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related security cogs: `tailgating` (companion cog at the same
  doorway — together they answer "who came through, and were they
  carrying?"), `perimeter-breach` (zone-level context that this cog's
  detection sits inside)
