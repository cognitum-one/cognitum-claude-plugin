# ADR: Spiking Tracker as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `spiking-tracker`
**Category**: ai

## Context

Tracking which "thing" in the room is moving — a person versus a fan versus a pet — is normally a Kalman filter or particle filter problem. Both work, but multi-target Kalman with data association is heavier than the seed needs, and particle filters thrash CPU on the Pi Zero 2 W. Spiking neural networks excel at this kind of low-power continuous tracking because they only compute when input arrives, not on every clock tick.

A leaky integrate-and-fire (LIF) population coding scheme lets the seed track 4–8 simultaneous targets with a few hundred neurons total, scaling cost with activity rather than time.

## Decision

Standalone armhf binary on Pi Zero 2 W. The cog runs an event-driven LIF network: 64 input neurons map CSI motion features into a 2D position grid, 32 hidden neurons learn to fire on persistent moving sources via spike-timing-dependent plasticity (STDP, A+=0.01, A-=0.012, τ=20 ms), and 8 output "tracks" use winner-take-all to bind to distinct sources.

State per neuron is `(membrane_v: f32, last_spike_us: u64)`. Membrane decays as `v *= exp(-Δt/τ)` only when input arrives — no per-tick update loop. Output JSON publishes (track_id, position, age_ms, confidence) per active track.

## Consequences

### Positive
- Event-driven compute scales with motion, not wall-clock; idle scenes use near-zero CPU.
- STDP learns target-specific filters online without labels.
- Naturally handles 4–8 simultaneous targets via WTA without explicit data association.

### Negative
- Tracks "swap" identity if two targets cross within the receptive field.
- STDP parameters are stiff to tune; bad settings produce silent or saturated networks.

### Neutral
- Refractory period and τ are exposed as flags for room-size tuning.

## Alternatives considered
- **Multi-target Kalman**: rejected — data-association cost grows quadratically with track count.
- **Particle filter**: rejected — constant-time per-particle update violates the < 5 % CPU budget.

## Plugin invocation
- `/spiking-tracker`
- `/spiking-tracker --once`
- `/spiking-tracker --console "<args>"`
- `/spiking-tracker --stop`
- `/spiking-tracker --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/spiking-tracker/`
- ADR-001
- `dtw-gesture-learn`, `pattern-sequence`
