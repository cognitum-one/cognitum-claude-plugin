# ADR: Person Matching as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `person-matching`
**Category**: signal

## Context

When more than one person is in the room, presence-only cogs collapse
them into a single occupant and lose useful information (who came in
first, did the same person leave that just arrived, are there two
people present right now). This cog is the de-anonymising layer over
the existing motion features. It is *not* identity recognition — it
matches **trajectories**, not faces or voices — and emits stable
session-local IDs (`person-1`, `person-2`, …) that reset on cog
restart.

Signal-processing approach: at each step extract a low-dimensional
"motion signature" from the 8-feature frame (variance + dominant-bin
shape + amplitude profile, projected to a 4D embedding via a fixed
random projection). Maintain up to `--max-people` (default 5) tracks,
each with an EMA of its signature. New frames are assigned to the
nearest track by Euclidean distance if within a gate; otherwise a new
track is opened. Tracks idle for > 60 s are marked `inactive` and may
be garbage-collected to make room for new IDs.

## Decision

Standalone armhf binary, reads `cog-sensor-sources` at 10 s default.
Pipeline: feature → 4D projection → nearest-track gate → assign-or-open
→ track update. JSON output: `active_people` (count), `assignment`
(current frame's track id), per-track `id`, `signature`, `last_seen`,
`session_seconds`, and a global `track_churn` indicator.

As Claude Code plugin: `/person-matching` wraps cog endpoints.

## Consequences

### Positive
- Disambiguates concurrent occupants without any identity-bearing data.
- Random-projection embedding is deterministic, fast, and has no
  training step.
- `max-people` cap bounds memory regardless of how many distinct
  signatures pass through.

### Negative
- Two people with very similar motion patterns will merge into one
  track (false negative).
- IDs do not survive a restart and are not portable across seeds.

### Neutral
- Confusable in highly cluttered scenes (party, crowded retail);
  designed for 2-5 occupants.

## Alternatives considered
- **Camera + face embedding.** Rejected: privacy and resource cost.
- **DBSCAN over a longer window.** Rejected: doesn't give online IDs;
  too slow for per-frame assignment.

## Plugin invocation
- `/person-matching` install, start, tail logs
- `/person-matching --once`
- `/person-matching --console "--max-people 8"`
- `/person-matching --stop`
- `/person-matching --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB (≤ 5 tracks × small state by default)
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/person-matching/`
- ADR-001 foundational
- Related cogs: `optimal-transport`, `hyperbolic-space`
