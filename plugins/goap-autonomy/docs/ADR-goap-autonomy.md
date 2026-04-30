# ADR: GOAP Autonomy as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `goap-autonomy`
**Category**: ai

## Context

The seed already detects, predicts, and ranks; what it does *not* do is decide. Goal-Oriented Action Planning (GOAP) is the right size of decision-maker for an embedded device — small enough to fit, structured enough to be auditable. Each action declares preconditions and effects as bitsets over a small world-state, and A* over action space finds the cheapest plan to satisfy a goal.

Reinforcement learning would also work but requires a reward model and training time the seed does not have; GOAP is fully specified and immediately interpretable.

## Decision

Standalone armhf binary on Pi Zero 2 W. World state is a 64-bit boolean vector (`PRESENCE_HOME`, `LIGHTS_ON`, `DOOR_LOCKED`, …). Actions are TOML records: `name`, `precond_mask`, `precond_value`, `effect_mask`, `effect_value`, `cost`. The planner runs IDA* (iterative deepening A*) with the unsatisfied-goal-bits Hamming distance as heuristic, capped at depth 12 and 5000 node expansions per cycle.

State machine: **idle → planning → executing (sequencing actions through cog endpoints) → succeeded/failed**. Failed plans roll the world model back from observation rather than the planned effect, so a missing actuator does not desync.

## Consequences

### Positive
- Plans are auditable — the user sees the action sequence before execution.
- Bitset state and Hamming heuristic make planning microsecond-fast at this scale.
- Action TOML is operator-editable without recompiling.

### Negative
- 64-bit state is enough for a small home; richer numeric state needs hierarchical decomposition.
- IDA* is depth-bounded; goals deeper than 12 actions silently fail.

### Neutral
- Actions are dispatched via existing cog plugin endpoints; the planner does not actuate hardware directly.

## Alternatives considered
- **Behaviour tree**: rejected — composes well but does not plan; rebuilds for new goals.
- **Q-learning**: rejected — needs a reward signal and warm-up the seed cannot provide reliably.

## Plugin invocation
- `/goap-autonomy`
- `/goap-autonomy --once`
- `/goap-autonomy --console "goal lights_off"`
- `/goap-autonomy --stop`
- `/goap-autonomy --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/goap-autonomy/`
- ADR-001
- `temporal-logic`, `meta-adapt`
