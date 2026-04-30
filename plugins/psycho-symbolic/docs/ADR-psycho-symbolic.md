# ADR: Psycho-Symbolic as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `psycho-symbolic`
**Category**: developer

## Context

Higher-level cogs (`goap-autonomy`, `meta-adapt`, scenario routines) need to reason over small structured knowledge — "if presence in kitchen and stove on then risk_unattended_cooking" — without pulling in a full LLM. The "psycho-symbolic" framing is shorthand for combining symbolic rules with sub-symbolic feature embeddings from the live stream, switching between three modes: forward chaining, backward chaining, and abductive (best-fit) inference.

This is a research cog. It is not a general theorem prover; it runs on the Pi Zero 2 W against a small in-memory rule set (≤ 200 rules) and the current 8-feature frame.

## Decision

`psycho-symbolic` loads a rule file (`/etc/cognitum/rules.psycho`), subscribes to `cog-sensor-sources`, and on each interval (default 10 s) runs one inference cycle in the configured mode. Symbolic predicates can match feature-stream conditions via threshold operators. Output is the derived facts and a confidence per fact (purely heuristic, derived from rule weight × match strength).

## Consequences

### Positive
- Keeps higher-level reasoning local; no cloud LLM call required.
- Three modes cover most plausible inference shapes for sensor reasoning.
- Rule file is human-editable and diffable.

### Negative
- Confidences are heuristic, not probabilistic — do not stack them as if they were.

### Neutral
- Hard rating reflects the breadth (forward + backward + abductive), not the per-cycle cost.

## Alternatives considered

- **Embed Prolog / miniKanren.** Rejected: footprint too large for default seed image.
- **Defer all reasoning to cloud LLM.** Rejected: latency, offline operation requirement, and ADR-069 phase gating.

## Plugin invocation
- `/psycho-symbolic` install, start, tail
- `/psycho-symbolic --once`
- `/psycho-symbolic --console "--interval 10"`
- `/psycho-symbolic --stop`
- `/psycho-symbolic --logs`

## Resource budget
- Binary: ~450 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/psycho-symbolic/` | ADR-001 | goap-autonomy | meta-adapt
