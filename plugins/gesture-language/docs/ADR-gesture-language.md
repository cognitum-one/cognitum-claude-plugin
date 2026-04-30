# ADR: Gesture Language as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `gesture-language`
**Category**: research

## Context

Sign-language recognition has historically required either a worn glove
(intrusive, costly) or a ceiling-mounted RGB camera (privacy concern,
not always available in the home). Research groups working on
deaf-accessibility prototypes need a passive sensor that can recognise a
limited vocabulary of canonical signs (yes / no / help / stop / numbers
0-9) at a distance of 1-3 metres without filming the signer.

A useful gesture recogniser on a Pi Zero 2 W must:

1. Run a small pre-trained gesture template bank against incoming CSI
   feature windows.
2. Tolerate variance in signing speed and handedness.
3. Be honest about its limitations: small vocabulary, single-signer at
   a time, no continuous-sentence parse.

## Decision

`gesture-language` matches the dynamic CSI feature window against a
template bank of canonical signs using normalised dynamic time warping.
At 10 s cadence it scans the last 3 s of features and emits the
best-match sign plus a similarity score. Output: `match` (sign id or
`null`), `score` (0-1), `template_count`, `last_match_ts`. The
template bank is shipped read-only with the cog; users add new signs
via a separate enrolment flow (`--enrol "sign-id"`).

As a Claude Code plugin, `/gesture-language` returns the live match and
the recent recognition log; `/gesture-language --console "--list"`
prints the template inventory.

## Consequences

### Positive
- No camera, no glove — research subjects sign naturally.
- DTW template matching is small, deterministic, and trainable per-user
  without a GPU.
- Honest scope: small canonical vocabulary, not full ASL parsing.

### Negative
- Single-signer assumption fails in group settings (the cog will pick
  the dominant motion source).

### Neutral
- Recognition latency is bounded by the matching window (default 3 s);
  shorter windows lose accuracy on slower signs.

## Alternatives considered

- **RGB-camera + transformer model.** Rejected on privacy and on Pi
  Zero 2 W RAM/CPU budget — quantised models still won't fit.
- **IMU-glove.** Complementary for ground-truth annotation in studies,
  rejected as the deployment form factor.

## Plugin invocation

- `/gesture-language` — install if needed, start, tail logs
- `/gesture-language --once` — current match snapshot
- `/gesture-language --console "<args>"` — arbitrary args (e.g. `--list`)
- `/gesture-language --stop` — stop
- `/gesture-language --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/gesture-language/`
- ADR-001 (foundational).
- `emotion-detect`, `music-conductor`.
