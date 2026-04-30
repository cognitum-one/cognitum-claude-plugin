# ADR: Music Conductor as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `music-conductor`
**Category**: research

## Context

Conducting research, music-education tooling, and accessible-conducting
interfaces need a way to read a conductor's tempo (beats per minute) and
dynamics (large-vs-small gesture amplitude) without putting markers on
the baton. Optical motion-capture systems exist (OptiTrack, Vicon) but
cost five figures and require a controlled stage. A seed sat at the
front of a rehearsal room can read the periodic vertical CSI modulation
of the baton arm and report tempo and amplitude in real time.

A useful conductor reader on a Pi Zero 2 W must:

1. Sample at a high enough rate (config `--sample-rate`, default
   100 Hz) to resolve beat onsets up to ~240 BPM.
2. Track tempo via autocorrelation on the dominant motion frequency.
3. Track dynamics via normalised gesture amplitude (large arc =
   forte cue; small wrist = piano).

## Decision

`music-conductor` samples motion at the configured `sample_rate` and
computes a 4-second autocorrelation each tick (default every 10 s, with
a faster inner streaming mode for live-rehearsal use). State machine:
`silent → counting-in → tempo-locked → dynamic-shift`. Output:
`bpm`, `bpm_confidence`, `dynamics_norm` (0-1), `state`,
`time_signature_hint` (3/4 vs 4/4 inferred from the autocorrelation
secondary peak), `last_beat_ts`.

As a Claude Code plugin, `/music-conductor` returns the live tempo,
dynamics, and a rehearsal log of tempo trajectory across the session.

## Consequences

### Positive
- Two orders of magnitude cheaper than mocap, with no markers on the
  baton.
- Tempo + dynamics together drive richer downstream tooling than tempo
  alone (a metronome cannot react to dynamics).
- 100 Hz default cleanly captures up to 240 BPM with margin.

### Negative
- Single-conductor assumption: a section leader cueing in parallel will
  confuse the autocorrelation.

### Neutral
- Sample rate is exposed because rehearsal rooms with strong RF noise
  benefit from a lower rate; the default is safe for most spaces.

## Alternatives considered

- **Optical mocap + IR markers.** Rejected on cost and stage-setup
  burden; remains the gold standard for offline analysis.
- **Wrist IMU on the baton wrist.** Rejected as intrusive — the whole
  point is to leave the conductor's gesture unchanged.

## Plugin invocation

- `/music-conductor` — install if needed, start, tail logs
- `/music-conductor --once` — current tempo snapshot
- `/music-conductor --console "<args>"` — arbitrary args
- `/music-conductor --stop` — stop
- `/music-conductor --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/music-conductor/`
- ADR-001 (foundational).
- `gesture-language`, `emotion-detect`.
