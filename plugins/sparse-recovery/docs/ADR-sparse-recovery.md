# ADR: Sparse Recovery as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `sparse-recovery`
**Category**: signal

## Context

The ESP32-to-seed UDP stream drops frames under WiFi congestion, brief
power dips, or when the seed is busy with another cog. Downstream cogs
that assume a regular cadence (time-crystal, person-matching) degrade
when gaps appear. The right name is "missing-data interpolation under a
sparsity prior" — but sparse-recovery is the marketing-friendly handle.

Signal-processing approach: the 8-feature stream is well-modelled as
sparse in a low-frequency cosine basis over short windows (most energy
sits in a handful of basis coefficients). When a frame is missing, we
solve a small per-channel least-squares fit to the surrounding
~30 frames using the lowest-frequency cosines as a basis, then
evaluate the fit at the missing timestamp. This is closer to a
DCT-truncation interpolant than to true compressive sensing — but it
gives the same practical recovery for the gap sizes we see (1-3
consecutive frames).

## Decision

Standalone armhf binary, reads `cog-sensor-sources`, publishes a
gap-filled parallel stream. Pipeline: 30-frame sliding window per
channel → detect missing timestamp(s) → DCT-basis least-squares fit
(K=4 basis terms) → evaluate at gap → emit reconstructed frame
flagged `recovered: true`. JSON sidecar reports `frames_recovered`,
`avg_gap_length`, `recovery_residual` (last-fit RMSE), and a
truthful `confidence` derived from gap length.

As Claude Code plugin: `/sparse-recovery` wraps cog endpoints.

## Consequences

### Positive
- Downstream cogs see a uniform-cadence stream and don't need
  gap-handling logic.
- Recovery quality scales with surrounding-window stability — quiet
  rooms fill in cleanly.
- Recovered frames are flagged so suspicious cogs can ignore them.

### Negative
- Long gaps (> 5 frames) are filled with low confidence; residual
  grows quickly.
- Cannot recover sharp transients that fall entirely inside the gap.

### Neutral
- The "sparse" prior is honest at this scale: we keep the 4 lowest
  cosines, which is what would dominate any L1 minimisation anyway.

## Alternatives considered
- **Linear interpolation.** Rejected: loses curvature on multi-frame
  gaps.
- **True L1 (basis-pursuit) recovery.** Rejected: solver cost
  unjustified at K=4 basis size.

## Plugin invocation
- `/sparse-recovery` install, start, tail logs
- `/sparse-recovery --once`
- `/sparse-recovery --console "--interval 5"`
- `/sparse-recovery --stop`
- `/sparse-recovery --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB (8 × 30-frame sliding windows)
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/sparse-recovery/`
- ADR-001 foundational
- Related cogs: `coherence-gate`, `temporal-compress`
