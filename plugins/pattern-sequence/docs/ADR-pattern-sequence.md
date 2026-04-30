# ADR: Pattern Sequence as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `pattern-sequence`
**Category**: ai

## Context

Daily routines — coffee at 7, gym at 6, lights off at 11 — are predictive signals other cogs can use ("alert if coffee is skipped two days running"). Detecting them needs frequent-sequence mining over a stream of discrete events. Apriori is too memory-heavy; a Variable-order Markov Model (VMM) using Prediction by Partial Matching (PPM) is the right size: it stores only sequences that actually occur, prunes by frequency, and predicts the next event with a confidence score.

This is a 60-second-interval cog because routines unfold over minutes-to-hours, not milliseconds.

## Decision

Standalone armhf binary on Pi Zero 2 W. The cog ingests discrete event tokens from `/run/cognitum/events.sock` (e.g. `light:on`, `door:closed`, `motion:kitchen`), feeds them into a depth-5 PPM trie capped at 8192 nodes, and at each interval emits (a) the top-K most frequent sequences of length ≥3, (b) a next-event prediction with confidence, and (c) any "broken" routine — a high-confidence prediction that did not occur.

State machine: **observing → predicting → mismatch (routine broken)**. Trie persists as `routines.cbor`; least-recently-used eviction keeps node count under cap.

## Consequences

### Positive
- Variable-order coverage means "coffee→commute→email" and "coffee→email" both contribute to learning.
- LRU eviction lets the trie operate indefinitely on bounded memory.
- Mismatch events are actionable: this is what makes other cogs care.

### Negative
- New routines need 5–10 occurrences before they stabilise, so first-week predictions are noisy.
- Token granularity is operator-defined; coarse tokens miss subtle patterns, fine tokens explode the trie.

### Neutral
- 60 s default interval matches routine timescales; can be lowered for faster events.

## Alternatives considered
- **Apriori frequent-itemset**: rejected — exponential candidate generation, no temporal order.
- **LSTM next-event prediction**: rejected — model size and training compute exceed Pi Zero 2 W budget.

## Plugin invocation
- `/pattern-sequence`
- `/pattern-sequence --once`
- `/pattern-sequence --console "predict"`
- `/pattern-sequence --stop`
- `/pattern-sequence --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/pattern-sequence/`
- ADR-001
- `temporal-logic`, `time-series-forecast`
