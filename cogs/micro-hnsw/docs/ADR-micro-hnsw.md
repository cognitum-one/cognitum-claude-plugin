# ADR: Micro HNSW as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `micro-hnsw`
**Category**: ai

## Context

Several cogs need "have I seen this before?" — fingerprint a CSI snapshot, classify an audio envelope, match a gesture template. Linear scan over a few hundred 64-dim vectors is fine; over a few thousand it dominates the cog's CPU budget. Hierarchical Navigable Small World (HNSW) is the established sub-millisecond approximate-NN structure, but stock implementations are heavy because they bundle persistence, multi-threading, and float32 SIMD that ARMv6 does not have.

The seed needs a stripped-down, single-threaded, integer-friendly HNSW that other cogs link or call as a service.

## Decision

Standalone armhf binary on Pi Zero 2 W. The cog exposes an HTTP add/search/delete API on the unix socket `/run/cognitum/micro-hnsw.sock`. Index parameters: M=16, efConstruction=80, efSearch=32, max 4096 vectors of dim ≤128. Distance is squared L2 over int8-quantised vectors, computed in u32 with a precomputed scale factor.

State machine: **building (insert) → ready (serving) → compacting (rebuilds layer-0 when deletion ratio > 25 %)**. The graph persists as a memory-mapped `index.bin` so cold-start is sub-50 ms.

## Consequences

### Positive
- Sub-millisecond search up to 4096 vectors at recall@10 ≥ 0.95.
- int8 quantisation cuts memory 4× versus float32 with negligible accuracy loss for this scale.
- Memory-mapped persistence amortises restart cost.

### Negative
- 4096-vector cap is firm; larger corpora need sharding or the full ruvector HNSW.
- Int8 quantisation hurts recall on very high-dim (>256) vectors.

### Neutral
- M, ef parameters expose a recall/latency dial but the defaults suit the seed's workload.

## Alternatives considered
- **Full ruvector HNSW**: rejected for this cog — too large for the 12 KB code budget.
- **Linear scan**: rejected — O(N) is fine at 100 vectors, painful at 4000.

## Plugin invocation
- `/micro-hnsw`
- `/micro-hnsw --once`
- `/micro-hnsw --console "search <vec>"`
- `/micro-hnsw --stop`
- `/micro-hnsw --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/micro-hnsw/`
- ADR-001
- `rag-local`, `dtw-gesture-learn`
