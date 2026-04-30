# ADR: Breathing Sync as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `breathing-sync`
**Category**: health

## Context

Respiratory entrainment between two co-sleeping or co-meditating
people is a well-documented phenomenon: the inter-person breathing
phase locks during deep relationship rapport, joint meditation, and
synchronous sleep cycles. The clinical use cases are niche but real
— couples therapy, mindfulness research, parent-infant bonding
biofeedback. The consumer use case is "couples want to see it."

The signal is **cross-correlation between two breathing-band
envelopes**, ideally separated spatially so a single seed can isolate
each person. On a Pi Zero 2 W with a single sensor source the
separation is imperfect — but a multi-occupant feature stream that
already exposes per-zone breathing channels (bed left / bed right)
makes the cog tractable.

Approach: bandpass each channel to 0.1–0.5 Hz, compute zero-lag
correlation over a rolling 30s window, and report a sync score in
[0, 1] plus a phase-lag estimate. Sustained correlation above a
threshold for a configurable dwell raises a `BREATHING_SYNCED` event.

## Decision

The cog runs as a standalone armhf binary on the seed, reading the
multi-channel `cog-sensor-sources` stream every `interval` seconds
(default 10s, range 1–3600s). It assumes two channels are present
and labelled; absent that, it gracefully reports `INSUFFICIENT_CHANNELS`
and exits with an error code so ops can correct the deployment.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/breathing-sync` that wraps the seed's cog management
endpoints (install, start, console, stop, logs).

## Consequences

### Positive
- Genuinely novel ambient signal — no consumer device today reports
  inter-person respiratory phase in real time.
- Pairs naturally with biofeedback UIs ("breathe with your partner")
  via the seed's existing console output.
- Cheap to run; cross-correlation on two short envelope windows is
  trivial work for the Pi Zero 2 W.

### Negative
- Requires the underlying sensor stack to spatially separate the two
  subjects — a single-channel deployment cannot run this cog
  meaningfully.
- The "sync score" is an instrument, not a relationship verdict; the
  cog must avoid leading users to draw psychological conclusions
  from a metric that wobbles with posture.

### Neutral
- Default 10s interval is biofeedback-fast; longer intervals work
  for research aggregation but feel laggy for live use.

## Alternatives considered

- **Worn chest straps for both subjects.** Rejected for v1: ruins the
  use case (couples don't strap on devices to fall asleep).
- **Single-channel inference.** Rejected: cannot separate two
  signals from one envelope without source-separation that busts
  the Pi Zero 2 W resource budget.

## Plugin invocation

- `/breathing-sync` — install if needed, start, tail logs
- `/breathing-sync --once` — one-shot console execution with `--once`
- `/breathing-sync --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--interval 5`)
- `/breathing-sync --stop` — stop the running cog
- `/breathing-sync --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/breathing-sync/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related cogs: `vital-trend` (shares the breathing-band bandpass
  front-end), `dream-stage` (consumes overnight sync trends as a
  proxy for paired sleep depth)
