# ADR: Rain Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `rain-detect`
**Category**: research

## Context

A seed mounted near a window, on a porch, or under an eave should know
when it's raining without a rain gauge. Rain has two acoustic
signatures the 8-feature ESP32 stream can pick up: a broadband, roughly
white noise floor that raises the variance channel, and a
characteristic mid-frequency spectral mass in the dominant-bin
features. The deployment target is hobby weather stations, garden
automations, and "did I leave the windows open" alerts.

Signal-processing approach: a two-feature classifier with hysteresis.
Compute the ratio of variance to amplitude (rain has high variance but
moderate amplitude — louder than silence, quieter than speech) and
gate it against a moving baseline. Confirm with a check that
dominant-bin energy is centred mid-band rather than low (HVAC) or
spiky-high (clinking, conversation). Intensity is mapped from the
log-scaled variance excess into a 0-3 scale (none, light, moderate,
heavy).

## Decision

Standalone armhf binary, reads `cog-sensor-sources` at 30 s default.
Pipeline: 5-min variance baseline → variance/amplitude ratio →
spectral-shape sanity check → hysteretic state machine **dry → onset →
raining (light/moderate/heavy) → easing → dry**. JSON output reports
the current state, intensity, minutes since onset, and confidence.

As Claude Code plugin: `/rain-detect` wraps cog endpoints.

## Consequences

### Positive
- No moving parts and no extra hardware — uses sensors the seed
  already has.
- Hysteresis avoids flapping during gusts and brief lulls.
- Three-tier intensity is enough for most automations (close window,
  pause sprinkler, alert).

### Negative
- Confuses sustained high-broadband sources (vacuum cleaner, large
  fan) for rain unless the spectral-shape check filters them.
- Indoor-only seeds with closed windows will miss everything but
  heavy rain.

### Neutral
- Requires a minute or two of dry baseline after boot before it
  reports confidently.

## Alternatives considered
- **Capacitive rain sensor input.** Rejected: defeats the point of an
  audio-only solution, adds wiring.
- **Trained acoustic classifier.** Rejected for v1 — model size budget
  is tight and the heuristic is good enough for state changes.

## Plugin invocation
- `/rain-detect` install, start, tail logs
- `/rain-detect --once`
- `/rain-detect --console "--interval 15"`
- `/rain-detect --stop`
- `/rain-detect --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB (5-min baseline + state)
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/rain-detect/`
- ADR-001 foundational
- Related cogs: `sound-classifier`, `coherence-gate`
