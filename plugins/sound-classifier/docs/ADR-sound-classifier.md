# ADR: Sound Classifier as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `sound-classifier`
**Category**: research

## Context

Home and small-business deployments want safety-relevant audio events —
glass break, smoke-alarm chirp, baby cry — surfaced as discrete events.
A full audio neural net is out of budget on a Pi Zero 2 W (RAM and
binary-size limits), so this cog uses a feature-template approach
against the 8-feature ESP32 stream rather than raw audio. The features
already encode broad spectral shape and onset characteristics, which is
enough for high-precision detection of a small, well-defined class set.

Signal-processing approach: each class has a hand-crafted signature in
the 8-feature space — glass break is a sharp onset with high-band
spectral mass and rapid decay; smoke-alarm chirp is a periodic
narrow-band burst at ~3 kHz mapped into the dominant-bin channel; baby
cry is sustained mid-band energy with prosodic variance. At each step
the cog scores the current frame (and a short window around it) against
each template via cosine similarity plus a temporal-shape penalty. The
top class above its per-class threshold fires.

## Decision

Standalone armhf binary, reads `cog-sensor-sources` at 5 s default
classification cadence (with internal sub-sampling for onset capture).
Pipeline: 4 s sliding window → per-class scorer → confidence threshold
→ JSON event with `class`, `confidence`, `onset_ts`, `duration_ms`.
Templates are embedded constants but overridable via a sidecar TOML.

As Claude Code plugin: `/sound-classifier` wraps cog endpoints.

## Consequences

### Positive
- High precision on the targeted small class set without an ML model.
- Templates are inspectable and editable — no opaque weights.
- Fits in budget with room for additional classes via TOML overrides.

### Negative
- Recall on out-of-template variants (different smoke-alarm models,
  unusual cry patterns) is lower than a trained classifier would give.
- Cannot distinguish similar-shape events (a clatter vs. a glass break
  on tiled floor) reliably without another sensor.

### Neutral
- A future v2 may swap in a quantized TFLite model when the budget
  permits; the JSON output schema is designed to be backend-agnostic.

## Alternatives considered
- **Quantized TFLite micro model.** Rejected for v1: binary-size and
  RAM cost too high alongside other cogs.
- **Threshold on amplitude only.** Rejected: zero precision on these
  classes.

## Plugin invocation
- `/sound-classifier` install, start, tail logs
- `/sound-classifier --once`
- `/sound-classifier --console "--interval 2"`
- `/sound-classifier --stop`
- `/sound-classifier --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB (4 s window + templates)
- CPU: < 5% of one core average

## See also
- Source: `cognitum-one/cogs:src/cogs/sound-classifier/`
- ADR-001 foundational
- Related cogs: `rain-detect`, `fall-detect`
