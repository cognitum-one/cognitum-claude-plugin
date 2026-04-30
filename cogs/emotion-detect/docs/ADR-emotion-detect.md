# ADR: Emotion Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `emotion-detect`
**Category**: research

## Context

Stress and arousal are observable in body language and breathing
envelope long before a subject reports them. Research labs studying
classroom attention, meditation efficacy, or therapy intake want a
non-contact, privacy-preserving stress proxy that does not require
fitting a chest band or wrist heart-rate monitor. Camera-based affective
computing is intrusive and bound up in IRB friction; a passive seed that
estimates breathing rate and postural micro-stillness from CSI features
is far less invasive.

A useful affective probe on a Pi Zero 2 W must:

1. Track breathing rate (typical 8-25 BrPM) from chest-cavity CSI
   modulation.
2. Track postural micro-stillness — fidget rate is a known stress
   correlate.
3. Emit an arousal score on a calm-stress axis without claiming to
   classify discrete emotions.

## Decision

`emotion-detect` extracts the breathing-band envelope (0.1-0.5 Hz) and
the fidget-band envelope (1-4 Hz) from `cog-sensor-sources`. At 10 s
cadence it composes a two-component arousal estimate: faster breathing
plus higher fidget = higher arousal. Output: `breathing_rate_bpm`,
`fidget_rate_hz`, `arousal_score` (0-1), `state` (`calm`, `neutral`,
`elevated`, `stressed`), `confidence`.

This is a research-grade proxy only — the cog explicitly does not
claim to identify discrete emotions (`happy`, `angry`, etc.); the
literature on doing that from sub-camera signals is not robust enough.

As a Claude Code plugin, `/emotion-detect` returns the live arousal
estimate and a session log suitable for export to study tooling.

## Consequences

### Positive
- No camera, no contact sensor — clears IRB review hurdles that block
  optical and wearable affective computing.
- Two-component decomposition (breathing + fidget) is more defensible
  than a black-box single score.

### Negative
- Coarse — meaningful at the session level (10s of seconds), not the
  individual gesture level.

### Neutral
- Per-subject calibration is recommended for the calm/elevated split —
  resting breathing rate varies widely.

## Alternatives considered

- **Webcam + facial-expression CNN.** Rejected: privacy, IRB friction,
  bias in expression-classifier ground truth.
- **Chest-band + PPG.** Complementary for ground truth in studies, but
  too invasive for naturalistic settings.

## Plugin invocation

- `/emotion-detect` — install if needed, start, tail logs
- `/emotion-detect --once` — current arousal snapshot
- `/emotion-detect --console "<args>"` — arbitrary args
- `/emotion-detect --stop` — stop
- `/emotion-detect --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/emotion-detect/`
- ADR-001 (foundational).
- `gesture-language`, `livestock-monitor`.
