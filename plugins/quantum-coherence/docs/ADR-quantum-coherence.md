# ADR: Quantum Coherence as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `quantum-coherence`
**Category**: developer

## Context

This is a research cog. Several upstream ruvector experiments use quantum-inspired representations (complex-valued embeddings, density-matrix smoothing, phase-coherent superposition) to model multi-channel sensor states. Those experiments need a runtime target inside the seed so we can replay against live data and compare against classical `coherence`.

There is no actual quantum hardware involved. "Quantum-inspired" means the math (unitary updates, density-matrix purity, off-diagonal magnitude) — not the hardware.

## Decision

`quantum-coherence` runs at 0.1 Hz over `cog-sensor-sources`, lifts the 8-feature frame into a complex amplitude vector, evolves it with a fixed unitary, and emits the off-diagonal magnitude of the rolling density matrix as a "quantum coherence" score. Output JSON includes `purity`, `off_diagonal_norm`, and a `classical_score` from sibling `coherence` for side-by-side evaluation.

## Consequences

### Positive
- Lets ruvector quantum-embedding work be evaluated on real Pi-Zero deployments.
- 16 KB binary stays inside the Pi Zero 2 W RAM budget.
- Output is comparable, by construction, to the classical `coherence` cog.

### Negative
- "Quantum" branding invites overclaim; the cog is a linear-algebra experiment, not physics.

### Neutral
- Hard difficulty rating reflects the math, not the runtime cost — CPU stays under 5%.

## Alternatives considered

- **Run the experiment on the host instead of the seed.** Rejected: we want apples-to-apples comparison against `coherence` on the same live stream.
- **Real quantum simulator (Qiskit).** Rejected: RAM and binary size.

## Plugin invocation
- `/quantum-coherence` install, start, tail
- `/quantum-coherence --once`
- `/quantum-coherence --console "--interval 10"`
- `/quantum-coherence --stop`
- `/quantum-coherence --logs`

## Resource budget
- Binary: ~440 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/quantum-coherence/` | ADR-001 | coherence | interference-search
