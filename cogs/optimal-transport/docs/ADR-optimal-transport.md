# ADR: Optimal Transport as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `optimal-transport`
**Category**: signal

## Context

Standard motion magnitude (frame-to-frame Euclidean distance over the
8 features) treats every channel as independent and over-reacts to a
single noisy channel. When motion in a room is real, multiple channels
shift in a *coordinated* way: a person crossing the room moves
amplitude, variance, and the dominant-bin centre together, in roughly
the same direction. Optimal-transport (specifically the 1D Wasserstein
distance over the channel-energy histogram) captures that
coordination, where Euclidean does not — at a small additional cost.

Signal-processing approach: treat the 8-channel frame as a discrete
mass distribution over channels (after softmax-normalisation of
absolute values). Frame-to-frame motion is the 1-Wasserstein
("earth-mover's") distance between consecutive distributions, computed
in O(n) via the cumulative-sum trick. We do *not* run a full Sinkhorn
solver — for an 8-bin distribution it would be overkill. Output is a
single scalar per step plus a coordination metric (cosine of the
direction vector with the previous direction vector).

## Decision

Standalone armhf binary, reads `cog-sensor-sources` at 10 s default.
Pipeline: previous-frame normalised distribution → cumulative-sum
1-Wasserstein → direction vector → coordination cosine. JSON output:
`emd` (the EMD distance), `coordination` (-1 to +1), `motion_class`
(`still`, `coordinated`, `noisy`) decided from the EMD/coordination
pair, and `samples_observed`.

As Claude Code plugin: `/optimal-transport` wraps cog endpoints.

## Consequences

### Positive
- Distinguishes real coordinated motion from single-channel noise
  spikes — the most common false-positive in the simpler cogs.
- O(n) cumulative-sum implementation is essentially free at n=8.
- Coordination metric is independently useful for downstream
  motion-quality decisions.

### Negative
- A single scalar can't localise *where* the motion is — that's still
  the job of person-matching or hyperbolic-space.
- The histogram interpretation of an 8-channel frame is heuristic;
  channels aren't truly bins of a physical density.

### Neutral
- Full Sinkhorn / 2D OT could be added later, but the cost-benefit at
  this dimensionality is poor.

## Alternatives considered
- **Frame-to-frame L2 distance.** Rejected: the very baseline this
  cog is meant to improve on.
- **Sinkhorn with regularisation.** Rejected for v1: solver overhead
  unjustified at n=8.

## Plugin invocation
- `/optimal-transport` install, start, tail logs
- `/optimal-transport --once`
- `/optimal-transport --console "--interval 5"`
- `/optimal-transport --stop`
- `/optimal-transport --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/optimal-transport/`
- ADR-001 foundational
- Related cogs: `person-matching`, `coherence-gate`
