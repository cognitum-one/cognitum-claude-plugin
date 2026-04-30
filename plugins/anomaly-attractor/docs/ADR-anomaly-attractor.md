# ADR: Anomaly Attractor as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `anomaly-attractor`
**Category**: ai

## Context

Per-sensor static thresholds are brittle: a temperature spike that's normal at 3 PM in July is anomalous at 3 AM in January. The seed needs an unsupervised "weirdness" detector that learns the joint distribution of its feature stream and flags points that fall outside it, without an offline training step.

Classic options (Isolation Forest, One-Class SVM) carry too much code. A 4-dimensional bottleneck autoencoder trained online with delta-rule SGD fits inside 10 KB and gives reconstruction error as a natural anomaly score — points the model has not seen well are reconstructed badly.

## Decision

Standalone armhf binary on Pi Zero 2 W. The `cog-sensor-sources` 8-feature stream feeds an 8-4-8 autoencoder with tanh activations. Weights live in a 96-float ring; updates are leaky (η=0.01, λ=1e-4) so the attractor "follows" slow seasonality but does not collapse onto an anomaly.

State machine: **warmup (first 200 samples populate baseline reconstruction stats) → tracking (running Welford mean/std of error) → ANOMALY (error > μ+3σ for ≥2 consecutive frames) → cooldown**. Output JSON includes the per-feature error vector so the user can see *which* signal went weird.

## Consequences

### Positive
- No training data needed; adapts to deployment in ~30 minutes.
- Per-feature error decomposition gives an interpretable "why".
- Leaky update prevents lock-in to a corrupted baseline.

### Negative
- Slow drifts that match the leakage rate are absorbed silently.
- A persistent anomaly eventually becomes the new normal.

### Neutral
- Z-score gate at 3σ is conservative; tune via `--threshold` for noisier environments.

## Alternatives considered
- **Isolation Forest**: rejected — tree memory and rebuild cost exceed the 10 KB budget.
- **Static z-score per channel**: rejected — ignores cross-feature correlation, which is most of what makes a point anomalous.

## Plugin invocation
- `/anomaly-attractor`
- `/anomaly-attractor --once`
- `/anomaly-attractor --console "<args>"`
- `/anomaly-attractor --stop`
- `/anomaly-attractor --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/anomaly-attractor/`
- ADR-001
- `time-series-forecast`, `ewc-lifelong`
