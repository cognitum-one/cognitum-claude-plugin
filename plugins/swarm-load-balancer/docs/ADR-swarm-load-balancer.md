# ADR: Swarm Load Balancer as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-load-balancer`
**Category**: swarm

## Context

Some cogs (HNSW search, MCP brain queries, federated vector lookups)
are CPU-bound on a Pi Zero 2 W. When a single seed serves many
clients — UI dashboards, automation rules, neighboring cogs — its load
average pegs and latency tails balloon. Spreading the work across
peers is the fix, but only if peer choice is dynamic, not round-robin.

This is an **infrastructure / control-plane cog** — it does not consume
sensor data. Its input is each peer's live load metrics from the
mesh; its output is a ranked routing table.

## Decision

`cog-swarm-load-balancer` runs an armhf binary on each seed. Every
`--interval` seconds (default 5) it exchanges load samples (CPU, RAM,
1-min request rate, recent latency p95) with `--peers` and recomputes
a weighted routing table that the local agent's HTTP layer consults
when forwarding queries. Peers in the red zone are temporarily evicted.
JSON output is `{peers: [{addr, weight, cpu_pct, p95_ms, status}],
self_weight}`.

Claude Code plugin: `/swarm-load-balancer` wraps the local
`/swarm/balance/*` endpoints — install, start, tail, force a recompute,
stop.

## Consequences

### Positive
- Hot peers shed load automatically; tail latency stays bounded.
- Same routing table works for any forwarded request type — search,
  inference, OTA fetch.

### Negative
- 5 s interval means a sudden spike on one peer can still cause
  one interval of bad routing before correction.

### Neutral
- Sticky routing for stateful sessions is **not** provided; clients
  needing affinity should pin manually.

## Alternatives considered

- **Round-robin DNS.** Rejected: ignores peer load and removes no
  unhealthy peer.
- **External HAProxy / nginx.** Rejected: extra infra and breaks
  air-gap operation.

## Plugin invocation

- `/swarm-load-balancer` install, start, tail
- `/swarm-load-balancer --once`
- `/swarm-load-balancer --console "--peers 169.254.42.2,169.254.42.3 --interval 3"`
- `/swarm-load-balancer --stop`
- `/swarm-load-balancer --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- `cognitum-one/cogs:src/cogs/swarm-load-balancer/`
- ADR-001 (cogs-as-plugins architecture)
- `swarm-cluster-monitor`, `swarm-distributed-store`
