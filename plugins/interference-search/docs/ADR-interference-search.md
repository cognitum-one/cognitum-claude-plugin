# ADR: Interference Search as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `interference-search`
**Category**: developer

## Context

Several seed cogs need to score many candidate hypotheses against the live feature stream — e.g. "which of N gesture templates fits the last 2 s," "which of M room layouts matches current presence pattern." Doing that as a serial loop wastes the small parallelism the Pi Zero 2 W actually has, and naive batch matching is dominated by memory traffic.

"Interference" here is a research framing borrowed from the quantum-inspired track: candidates are summed coherently and the surviving (non-cancelling) modes are read out. In practice it reduces to a batched cosine-similarity sweep with a shaped weighting kernel.

## Decision

`interference-search` accepts a candidate set on stdin (newline-delimited JSON, ≤ 256 candidates of ≤ 64 floats each) and a query window from `cog-sensor-sources`. It emits the top-K candidates with scores. Inner loop is a tight batched dot-product against a rolling buffer; the "interference" overlay is a Hann window applied to the candidate side. Default interval 10 s.

## Consequences

### Positive
- One reusable matcher for any cog that needs "best-of-N templates."
- 14 KB binary; the matrix multiply is the only real work.
- Composable: caller decides what counts as a "candidate."

### Negative
- Hard cap of 256 candidates. Larger search spaces need an HNSW index, not this cog.

### Neutral
- The quantum framing is presentation; the code is a windowed cosine sweep.

## Alternatives considered

- **HNSW (`micro-hnsw` cog) for the same role.** Rejected for small (< 256) sets — index overhead exceeds linear scan.
- **Per-cog inline matchers.** Rejected: duplication and inconsistent ranking.

## Plugin invocation
- `/interference-search` install, start, tail
- `/interference-search --once`
- `/interference-search --console "--interval 10"`
- `/interference-search --stop`
- `/interference-search --logs`

## Resource budget
- Binary: ~430 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/interference-search/` | ADR-001 | quantum-coherence | micro-hnsw
