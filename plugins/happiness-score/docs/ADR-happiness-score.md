# ADR: Happiness Score as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `happiness-score`
**Category**: research

## Context

Eldercare facilities, dorms, and shared workspaces want a *non-invasive*
"is the room doing OK?" indicator without cameras or wearables. The
honest framing: this cog produces a heuristic well-being proxy, not a
clinical mood score. It combines three readily-available signals from
the 8-feature ESP32 stream — overall motion energy (more activity ≈
healthier baseline), motion regularity (smooth daily rhythms ≈ stable
routines), and acoustic prosody proxy (mean and slope of voiced-band
energy bursts, as a coarse stand-in for "voices sound lively rather
than flat or shouty").

Signal-processing approach: three sub-scores in [0,1], blended with
fixed weights and exponentially smoothed over a 6-hour window. Each
sub-score is a sigmoid of the corresponding feature against a
deployment-tuned midpoint. The output is a 0-100 number plus the three
sub-scores so the user can see *why* the score moved.

## Decision

Standalone armhf binary, reads `cog-sensor-sources` at 10 s default.
Pipeline: per-frame feature extraction → three sub-score sigmoids →
weighted blend → 6-hour EMA. JSON output: `score` (0-100), `activity`,
`rhythm`, `prosody` (each 0-1), `samples_observed`, `trend` (one of
`improving`, `stable`, `declining` from a 24 h linear fit on the
score).

As Claude Code plugin: `/happiness-score` wraps cog endpoints.

## Consequences

### Positive
- No camera, no wearable, no microphone recording — only derived
  features.
- Sub-scores are visible, so the score is auditable rather than a
  black box.
- 6-hour EMA prevents the score whipsawing between events.

### Negative
- It is a *heuristic*. It correlates with well-being only loosely and
  must not be used for clinical decisions.
- Empty rooms score low on activity and rhythm by design — the cog
  cannot distinguish "no one home" from "everyone is sad".

### Neutral
- The fixed weights are deliberately conservative; tuning per-deployment
  is supported but not required.

## Alternatives considered
- **Camera-based affect recognition.** Rejected: privacy, cost, and
  dubious accuracy.
- **Self-report integration.** Rejected for v1: outside the seed's
  on-device scope.

## Plugin invocation
- `/happiness-score` install, start, tail logs
- `/happiness-score --once`
- `/happiness-score --console "--interval 30"`
- `/happiness-score --stop`
- `/happiness-score --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/happiness-score/`
- ADR-001 foundational
- Related cogs: `plant-growth` (similar EMA pattern), `sound-classifier`
