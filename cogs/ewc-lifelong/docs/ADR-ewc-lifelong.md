# ADR: EWC Lifelong as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `ewc-lifelong`
**Category**: ai

## Context

A seed deployed for 6+ months sees regime shifts: a new tenant, a season change, a remodelled room. Naive online learners catastrophically forget the previous regime; full replay buffers exhaust the SD card. Elastic Weight Consolidation (EWC) is the standard mitigation — it computes a diagonal Fisher Information matrix per task and adds a quadratic penalty pulling important weights back to their consolidated values.

For the seed, EWC fits because Fisher diagonals are O(n) in parameters, not O(n²), and a small online network has only a few hundred weights. This makes lifelong adaptation tractable in 8 KB of cog code.

## Decision

Standalone armhf binary on Pi Zero 2 W. The cog hosts a tiny MLP (16 → 8 → 4) trained on the `cog-sensor-sources` stream with a self-supervised next-step prediction objective. On a `/consolidate` API call (or automatic time/drift trigger) the running squared-gradient estimate becomes the new Fisher diagonal `F_i`, and the loss switches to `L_new + λ Σ F_i (θ_i − θ*_i)²` with λ=0.4.

State machine: **learning → consolidating → protected**, with up to 4 stacked tasks. Per-task Fisher and parameter snapshots persist in `tasks.cbor`.

## Consequences

### Positive
- Old regimes survive new training without storing raw history.
- Diagonal Fisher keeps memory linear; 4 stacked tasks fit in < 4 KB.
- The same binary handles arbitrarily many sequential tasks via FIFO stacking.

### Negative
- λ tuning is delicate — too high freezes learning, too low forgets.
- Diagonal Fisher ignores cross-parameter dependencies and underestimates importance for correlated weights.

### Neutral
- "Task boundary" is operator-defined; auto-detection is best-effort.

## Alternatives considered
- **Replay buffer**: rejected — SD wear and storage cost over months.
- **Naive fine-tuning**: rejected — catastrophic forgetting on month-scale drift.

## Plugin invocation
- `/ewc-lifelong`
- `/ewc-lifelong --once`
- `/ewc-lifelong --console "consolidate"`
- `/ewc-lifelong --stop`
- `/ewc-lifelong --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/ewc-lifelong/`
- ADR-001
- `meta-adapt`, `anomaly-attractor`
