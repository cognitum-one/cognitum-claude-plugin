# ADR: Hyperbolic Space as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `hyperbolic-space`
**Category**: research

## Context

Hierarchical relationships — room A is part of floor 2 is part of
building X — embed badly in flat Euclidean space; the exponential
volume of a hyperbolic ball matches the exponential branching of a
tree. This cog is the seed-side companion to ruvector's hyperbolic
embedder: it takes the 8-feature ESP32 frame and projects it into a
2D Poincaré disk so a UI or downstream cog can visualise where the
current room/state sits relative to a learned tree of states.

Signal-processing approach: maintain a small set of "anchor" frames
captured during onboarding (one per labelled location/state). At each
step, compute distances to anchors, then place the current frame on
the disk via a Sarkar-style embedding update — closer anchors pull
the point inward, farther anchors push it outward, with the metric
contracted by the Poincaré factor near the boundary.

## Decision

Standalone armhf binary, reads `cog-sensor-sources` at 10 s default.
Pipeline: anchor table (loaded from disk; updatable via console) →
distance vector → Poincaré-disk coordinate update → JSON output with
`u`, `v` (disk coords, |·|<1), `nearest_anchor`, `tree_depth_proxy`
(from `-log(1-|p|^2)`), and `confidence`.

As Claude Code plugin: `/hyperbolic-space` wraps cog endpoints.

## Consequences

### Positive
- Hierarchical structure (rooms within floors) is preserved with
  small distortion compared to a 2D Euclidean projection.
- Output is bounded (`|p| < 1`) and ready to plot directly.
- Anchor set is small (typically 5-15 entries), keeping RAM tiny.

### Negative
- Quality is bounded by the anchor set; bad onboarding gives a
  meaningless disk.
- Two-dimensional projection still loses information for very deep
  hierarchies.

### Neutral
- Numerically careful near the boundary (|p| → 1); we cap at 0.999.

## Alternatives considered
- **Euclidean t-SNE / UMAP.** Rejected: distortion grows fast for
  hierarchical data, and per-step UMAP isn't tractable on Pi Zero.
- **Higher-dimensional hyperbolic embedding.** Rejected for v1: 2D is
  enough for the visualisation use case.

## Plugin invocation
- `/hyperbolic-space` install, start, tail logs
- `/hyperbolic-space --once`
- `/hyperbolic-space --console "--interval 5"`
- `/hyperbolic-space --stop`
- `/hyperbolic-space --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB (anchor table + per-step state)
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/hyperbolic-space/`
- ADR-001 foundational
- Related cogs: `person-matching`, `optimal-transport`
