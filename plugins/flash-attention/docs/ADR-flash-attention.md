# ADR: Flash Attention as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `flash-attention`
**Category**: signal

## Context

The ESP32 frame has 8 channels; not all of them are equally informative
at any given moment. Downstream cogs (sound-classifier, fall-detect,
person-matching) sometimes spend cycles on noisy channels that
contribute nothing. The "flash attention" name is inherited from the
Tri Dao kernel, but at this scale and on this CPU we are not running
softmax over a thousand keys — we are running a much smaller and much
honest top-K channel selector with a temporal-decay weighting.

Signal-processing approach: at each step compute a per-channel
"importance" as the running variance times the absolute deviation from
the channel's slow EMA. Pick the top `--top-k` channels (default 4) and
emit a masked frame in which the non-selected channels are zeroed and
flagged. Other cogs that subscribe to the flash-attention output
process less data and run faster; cogs that subscribe to the raw
stream are unaffected.

## Decision

Standalone armhf binary, reads `cog-sensor-sources`, publishes a
parallel attended stream on a local socket. Pipeline: per-channel EMA
+ variance → importance score → top-K selection → masked-frame
publication. JSON sidecar reports the selected channel indices,
their importance scores, and a stability metric (how often the same
top-K is chosen across a 1-min window).

As Claude Code plugin: `/flash-attention` wraps cog endpoints.

## Consequences

### Positive
- Downstream cogs that opt in see less work per frame.
- Selection is data-driven, so noisy channels drop out automatically.
- Stability metric makes it observable when the room/scene changes.

### Negative
- Adds a small latency (one frame) to anything subscribing through
  flash-attention rather than the raw stream.
- Top-K is hard cutoff; a borderline 5th channel is fully zeroed.

### Neutral
- Memory-resident only — there is no persistent attention state across
  reboots, by design.

## Alternatives considered
- **Per-cog channel selection.** Rejected: duplicated logic in every
  consumer, and they don't share a baseline.
- **Soft (continuous) channel weights.** Rejected for v1: doesn't give
  the bandwidth savings, just changes the math.

## Plugin invocation
- `/flash-attention` install, start, tail logs
- `/flash-attention --once`
- `/flash-attention --console "--top-k 6"`
- `/flash-attention --stop`
- `/flash-attention --logs`

## Resource budget
- Binary: 400-460 KB stripped armhf
- RAM: < 2 MB
- CPU: < 5% of one core

## See also
- Source: `cognitum-one/cogs:src/cogs/flash-attention/`
- ADR-001 foundational
- Related cogs: `coherence-gate`, `sparse-recovery`
