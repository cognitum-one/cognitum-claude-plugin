# ADR: Meta Adapt as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `meta-adapt`
**Category**: ai

## Context

Every other cog on the seed has hyperparameters — DTW match threshold, autoencoder learning rate, fall-detect cooldown. Operators get them wrong, and the only feedback signal is "false positives are noisy" or "I missed an event." A meta-controller that nudges these knobs based on observed downstream reward is the sweet spot between hand-tuning and full Bayesian optimisation, which would not fit.

The decision is to ship a small online tuner using Simultaneous Perturbation Stochastic Approximation (SPSA): it estimates the reward gradient with two perturbed evaluations per step regardless of dimensionality, which is ideal for tuning 3–6 knobs cheaply.

## Decision

Standalone armhf binary on Pi Zero 2 W. The cog reads sibling cog stats from `/run/cognitum/cogs/*/stats.json`, computes a configurable reward (e.g. `precision − 0.3·rate`), perturbs each registered knob by ±c, runs the sibling cog for one interval per perturbation, and applies an SPSA gradient step (a=0.05, c=0.1 with k^-0.602/k^-0.101 decay).

State: **observe → perturb+ → perturb- → step → settle**, persisted as `state.cbor`. A trust-region clamp prevents step sizes greater than 20 % of the parameter range.

## Consequences

### Positive
- Two reward evaluations per cycle scales to many knobs without explosion.
- All updates are bounded — never moves a knob by more than 20 % per cycle.
- Tunes any cog that exports a stats endpoint; no per-cog code.

### Negative
- SPSA is noisy; convergence takes 50–200 cycles.
- Misaligned reward functions optimise the wrong thing convincingly.

### Neutral
- The reward expression lives in user config and is the operator's responsibility.

## Alternatives considered
- **Bayesian optimisation**: rejected — Gaussian Process memory grows quadratically and exceeds budget.
- **Grid search**: rejected — combinatorial blow-up beyond two knobs.

## Plugin invocation
- `/meta-adapt`
- `/meta-adapt --once`
- `/meta-adapt --console "<args>"`
- `/meta-adapt --stop`
- `/meta-adapt --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/meta-adapt/`
- ADR-001
- `ewc-lifelong`, `goap-autonomy`
