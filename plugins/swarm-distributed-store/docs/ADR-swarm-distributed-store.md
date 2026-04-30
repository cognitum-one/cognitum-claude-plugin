# ADR: Distributed Vector Store as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-distributed-store`
**Category**: swarm

## Context

A single Pi Zero 2 W has 512 MB RAM and a finite HNSW capacity. Once a
fleet collects enough embeddings — household behavioral profiles, mesh
fingerprints, witnessed events — no single seed can hold the working
set. Operators need a way to **shard vectors across the mesh** and run
a single query that fans out and merges results.

This is an **infrastructure / control-plane cog** — it consumes mesh
state, not sensor data. Its input is the AgentDB shard map and peer
HNSW indices; its output is the union of top-k neighbors across peers.

## Decision

`cog-swarm-distributed-store` runs an armhf binary on each participating
seed. On the configured interval (default 10 s) it exchanges shard
manifests with the peers in `--peers`, rebalances ownership using
consistent hashing, and exposes a federated-search RPC. JSON output
reports `{shard_count, owned_keys, replicated_keys, peer_lag_ms[]}`.

Claude Code plugin: `/swarm-distributed-store` wraps the local
`/swarm/store/*` endpoints — install, start, tail, force a rebalance,
and stop.

## Consequences

### Positive
- Working set scales linearly with mesh size, not per-seed RAM.
- Replication factor (default 2) tolerates one peer offline without
  query loss.
- Falls back to local-only mode if `--peers` is empty.

### Negative
- Federated query latency is gated by the slowest peer; one
  unreachable seed adds a timeout per query.

### Neutral
- Consistent hashing assumes stable peer IDs. Frequent peer churn
  triggers more rebalances and more delta traffic.

## Alternatives considered

- **Single primary + replicas.** Rejected: primary becomes a hot spot
  and a SPOF on a fleet that may partition.
- **External vector DB (e.g. Qdrant on a server).** Rejected: defeats
  the air-gappable, self-contained seed promise.

## Plugin invocation

- `/swarm-distributed-store` install, start, tail
- `/swarm-distributed-store --once`
- `/swarm-distributed-store --console "--peers 169.254.42.2,169.254.42.3 --interval 5"`
- `/swarm-distributed-store --stop`
- `/swarm-distributed-store --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- `cognitum-one/cogs:src/cogs/swarm-distributed-store/`
- ADR-001 (cogs-as-plugins architecture)
- `swarm-delta-sync`, `swarm-load-balancer`
