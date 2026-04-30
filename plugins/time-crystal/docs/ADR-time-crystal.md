# ADR: Time Crystal as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `time-crystal`
**Category**: research

## Context

The "time crystal" name is borrowed from condensed-matter physics, but
the cog itself does something concrete and modest: it detects periodic
structure in the 8-feature ESP32 stream and reports the dominant period
and its phase stability over time. The use case is room-occupancy
rhythms (HVAC cycles, foot-traffic loops, washing-machine cycles) and
classroom demos of frequency-domain analysis.

Signal-processing approach: maintain a 1024-sample ring (~3 hours at
10 s default cadence) of one selected channel — usually the variance
channel because it has the cleanest periodicities. Run a Goertzel-style
narrowband DFT scan over a small set of candidate periods (1 min to
~90 min) once per minute. The strongest peak above a noise floor is
reported as the "crystal"; phase stability is the autocorrelation of
peak phase across consecutive scans. No FFT library — the candidate set
is small enough that direct evaluation is cheaper.

## Decision

Standalone armhf binary, reads `cog-sensor-sources` at 10 s default.
Pipeline: ring buffer → per-period Goertzel → peak detection → phase
tracker. Emits JSON with `period_seconds`, `amplitude`,
`phase_stability` (0-1), `samples_observed`, and `is_locked` (true once
phase stability stays above 0.6 for three consecutive scans).

As Claude Code plugin: `/time-crystal` wraps cog endpoints.

## Consequences

### Positive
- Surfaces real, useful periodicities (HVAC, traffic) the user
  otherwise wouldn't see.
- Direct Goertzel keeps the binary small — no FFT dependency.
- Output is human-readable: "39-minute cycle, phase-locked".

### Negative
- ~3-hour warm-up before the longest candidate period is observable.
- Misses non-stationary periodicities (drifting frequency).

### Neutral
- The poetic name is intentional and stays. The output is
  unambiguous about what is actually measured.

## Alternatives considered
- **Full FFT every minute.** Rejected: too much work for the ARM core
  and we only care about a small candidate set.
- **Wavelet decomposition.** Rejected: bigger binary, more state, no
  real benefit for these timescales.

## Plugin invocation
- `/time-crystal` install, start, tail logs
- `/time-crystal --once`
- `/time-crystal --console "--interval 30"`
- `/time-crystal --stop`
- `/time-crystal --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB (1024-sample ring + small Goertzel state)
- CPU: < 5% of one core (peak ~10% during the per-minute scan)

## See also
- Source: `cognitum-one/cogs:src/cogs/time-crystal/`
- ADR-001 foundational
- Related cogs: `coherence-gate`, `temporal-compress`
