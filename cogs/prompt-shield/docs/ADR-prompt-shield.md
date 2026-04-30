# ADR: Prompt Shield as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `prompt-shield`
**Category**: security

## Context

Seeds accept commands and sensor-derived prompts from a mesh of peers,
upstream agents, and embedded LLM cogs. Two attack classes are common:
**replay** (a captured-and-resent legitimate command) and **injection**
(adversarial text smuggled inside a sensor description, OTA hint, or
peer broadcast that hijacks downstream LLM cogs). Neither is caught by
TLS — both ride inside legitimate transports. Cloud-side filtering is
not viable; the seed is offline-capable and must defend itself.

The relevant signal-processing approach is **temporal hash deduplication**
(rolling Bloom filter of recent command digests, rejecting near-duplicates
within a sliding window) combined with **token-entropy thresholding** on
free-text payloads. High-entropy or improbable n-gram sequences mark
injection attempts; low-novelty repeats inside the window mark replays.

## Decision

Standalone armhf binary on seed. Sits in front of cog ingestion at 2 s
scan cadence. Maintains a 256-bit rolling Bloom over a 5 min window of
SHA-256(command || nonce) and runs a tiny char-bigram entropy estimator
on free-text fields. State machine:
`clean → suspicious → BLOCKED`. Emits JSON with `verdict`, `reason`,
`entropy_bits`, `replay_distance_ms`. Packaged as Claude Code plugin:
slash command `/prompt-shield` wraps seed's cog management endpoints.

## Consequences

### Positive
- Operates entirely on-device — no cloud round trip per command.
- Replay window is tunable per deployment (5 min default, configurable).
- Catches both repeat-attack and novel-injection without ML overhead.

### Negative
- Bloom filter false-positive rate trades RAM for accuracy; 5 min × 256 b
  budget admits ~0.1% legitimate replays misclassified.
- Cannot defend against semantic-equivalent rewrites of an attack.

### Neutral
- Entropy threshold is corpus-dependent — English vs. structured JSON
  payloads have different baselines.

## Alternatives considered

- **Cloud LLM-based prompt firewall.** Rejected: latency, offline gap,
  cost. Could be a v2 advisory layer.
- **Strict allowlisting.** Rejected: brittle for a self-learning fleet
  whose command surface evolves.

## Plugin invocation

- `/prompt-shield` — install if needed, start, tail logs
- `/prompt-shield --once` — one-shot via `/console` with `--once`
- `/prompt-shield --console "<args>"` — arbitrary args
- `/prompt-shield --stop` — stop cog
- `/prompt-shield --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/prompt-shield/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `fleet-auth`, `audit-logger`.
