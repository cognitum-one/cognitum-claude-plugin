# ADR: Health Monitor as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `health-monitor`
**Category**: health

## Context

The seed already ships specialized health cogs: `cardiac-arrhythmia`, `breathing-sync`, `fall-detect`, `dream-stage`. A user installing the seed for an aging parent does not want to wire up four cogs and reconcile their outputs. They want one pane: heart rate, breathing rate, sleep quality, fall alerts.

`health-monitor` is the composer. It does not re-derive any signal; it subscribes to the specialist cogs and emits a single rolled-up health JSON.

## Decision

`health-monitor` runs at 0.1 Hz (default 10 s), reads the latest published events from `cardiac-arrhythmia`, `breathing-sync`, `fall-detect`, and `dream-stage` via the seed event bus, and emits a unified record: `{ heart_rate, breathing_rate, sleep_stage, last_fall, alerts[] }`. If a source cog has not published in `staleness_window_secs`, that field is `null` and an alert `"<cog> stale"` is added — never fabricated values.

## Consequences

### Positive
- One endpoint for caregiver UIs; no need to know the cog topology.
- Featured cog — primary surface for the "ambient health" use case.
- Staleness handling prevents the fan-in from masking upstream failures.

### Negative
- Adds a hop of latency vs subscribing to the source cogs directly.

### Neutral
- Composes existing cogs; depends on them being installed and running. The cog will start with degraded output if any are missing.

## Alternatives considered

- **Re-derive every signal in one big cog.** Rejected: would duplicate the specialists and explode the binary.
- **Cloud-side composition.** Rejected: offline operation is a core seed contract.

## Plugin invocation
- `/health-monitor` install, start, tail
- `/health-monitor --once`
- `/health-monitor --console "--interval 10"`
- `/health-monitor --stop`
- `/health-monitor --logs`

## Resource budget
- Binary: ~440 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/health-monitor/` | ADR-001 | cardiac-arrhythmia | breathing-sync | fall-detect (ADR-002)
