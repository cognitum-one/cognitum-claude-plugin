# ADR: Federated Learning as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `federated-learning`
**Category**: ai

## Context

Multiple seeds in a household, office, or fleet learn similar things — gait baselines, anomaly attractors, routine vocabularies — but each only sees its own slice. Pooling raw data to a cloud trainer is unacceptable on privacy and bandwidth grounds. Federated averaging (FedAvg) is the canonical answer: each seed trains locally on its data and only model deltas leave the device. Secure aggregation can be added later, but for v1 plain-text deltas over the existing mesh tunnel are sufficient.

The seed already runs an authenticated UDP mesh (ADR-084), which gives a cheap transport for FedAvg rounds without provisioning a separate channel.

## Decision

Standalone armhf binary on Pi Zero 2 W. The cog wraps a sibling local model (anomaly-attractor, ewc-lifelong, etc.) registered via `/run/cognitum/fed/<peer-cog>.sock`. Each round (default 30 s) the cog: (1) snapshots local weights `w_i`, (2) trains for `local_epochs` on locally-buffered samples, (3) computes delta `Δ_i = w_i_new − w_i`, (4) gossips Δ to mesh peers via UDP-tunnel multicast, (5) waits up to `agg_timeout`, (6) averages received deltas weighted by reported sample count, (7) applies the average to local weights.

State machine: **local-train → broadcasting → aggregating → applying → idle**. Deltas are int8-quantised with stochastic rounding to keep packet sizes under one MTU per round.

## Consequences

### Positive
- Raw data never leaves the seed; only quantised weight deltas do.
- Reuses the existing encrypted mesh tunnel — no new ports, no new keys.
- Quantised deltas keep one round under 1500 bytes for typical model sizes.

### Negative
- Plain-text deltas (within the encrypted tunnel) are vulnerable to a malicious peer reconstructing training data via gradient inversion. Secure aggregation is a future ADR.
- Heterogeneous data across peers (non-IID) slows convergence and can reduce accuracy.

### Neutral
- Round cadence is operator-tuned — frequent rounds adapt fast but cost mesh bandwidth.

## Alternatives considered
- **Centralised training in the cloud**: rejected — privacy and bandwidth.
- **Split learning**: rejected — requires a coordinator with always-on connectivity, which the mesh is not.

## Plugin invocation
- `/federated-learning`
- `/federated-learning --once`
- `/federated-learning --console "peers"`
- `/federated-learning --stop`
- `/federated-learning --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/federated-learning/`
- ADR-001
- `ewc-lifelong`, `pagerank-influence`
