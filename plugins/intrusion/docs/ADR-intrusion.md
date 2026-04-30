# ADR: Intrusion Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `intrusion`
**Category**: security

## Context

The cheap reed switches and PIR sensors in mass-market alarm panels
fail in two predictable ways: they false-positive on cats and ceiling
fans, and they miss a determined intruder who knows to step around the
PIR cone. An ambient feature-stream-based detector can do better
because it sees the whole room as a single physical state, not a few
discrete zones.

Target deployment is residential and small-commercial overnight /
when-away monitoring. The signal of interest is **any deviation from
the empty-room baseline** beyond a z-score threshold, sustained for
multiple consecutive frames. The "arm after baseline learning" gate
prevents the obvious failure mode: arming the system while you're
still in the room.

Approach: at arm-time, accumulate a baseline of motion / amplitude
statistics for `arm_after` seconds (default 60s). After arm, every
frame is z-scored against that baseline; `trigger_count` consecutive
frames above `detection_threshold` (default 3.0σ) emits the alarm.

## Decision

The cog runs as a standalone armhf binary on the seed, reading
`cog-sensor-sources` every `interval` seconds (default 3s). It owns a
small Disarmed → Arming → Armed → Triggered state machine and emits
structured `INTRUSION_DETECTED` JSON on alarm. The `trigger_count`
gate (default 2 consecutive) is the single most important false-
positive suppressor.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/intrusion` that wraps the seed's cog management endpoints
(install, start, console, stop, logs).

## Consequences

### Positive
- Whole-room awareness rather than discrete zone tripwires; harder to
  defeat by edge-walking or low-crawling than PIR.
- Per-deployment self-baselining means the same binary works in a
  studio apartment and a warehouse without code change.
- The `trigger_count` consecutive-frame gate kills the dominant false-
  positive mode (single-frame sensor glitches).

### Negative
- HVAC start-up causes a sustained baseline shift; the cog re-baselines
  on disarm but a sustained-on cycle during arm-time will drift.
- Cannot identify the intruder — that's `weapon-detect` or a camera's
  job, deliberately out of scope here.

### Neutral
- Default 3.0σ threshold is conservative; sensitive deployments lower
  to 2.0σ at the cost of more false positives, paranoid deployments
  raise to 4.0σ.

## Alternatives considered

- **PIR-only detection.** Rejected: PIR's blind spots are the entire
  reason this cog exists.
- **Cloud ML on raw audio.** Rejected: privacy hostile and the
  z-score-on-baseline approach catches the high-yield cases at a
  fraction of the resource cost.

## Plugin invocation

- `/intrusion` — install if needed, start, tail logs
- `/intrusion --once` — one-shot console execution with `--once`
- `/intrusion --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--detection-threshold 2.5`, `--arm-after 120`)
- `/intrusion --stop` — stop the running cog
- `/intrusion --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/intrusion/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related security cogs: `perimeter-breach` (multi-zone version of
  this single-zone pattern), `loitering` (post-intrusion dwell-time
  enrichment)
