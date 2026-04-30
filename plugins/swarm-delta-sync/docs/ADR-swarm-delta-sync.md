# ADR: Swarm Delta Sync as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-delta-sync`
**Category**: swarm

## Context

A fleet of Cognitum seeds accumulates local state — vector embeddings,
event logs, witness chain entries, learned cog parameters — at very
different rates. Naively replicating full snapshots between peers
saturates the WiFi mesh and burns Pi Zero 2 W flash. Operators need a
control-plane primitive that continuously reconciles peers while sending
**only changed bytes**.

This is an **infrastructure / control-plane cog** — it does not read
sensor data. Its input is the local AgentDB delta log and the peer's
remote delta log; its output is a sync transcript.

The candidate alternatives were a manual `rsync` cron, a heavyweight
Raft cluster, and full snapshot replication. Each loses on either
operator ergonomics, RAM headroom, or bandwidth.

## Decision

Ship `cog-swarm-delta-sync` as an armhf binary on the Pi Zero 2 W. It
runs on a configurable interval (default 60 s) and exchanges Merkle-style
delta hashes with `--peer` over the encrypted mesh tunnel (ADR-084).
Only diverging branches are pulled. JSON output reports
`{peer, sent_bytes, received_bytes, deltas_applied, lag_ms}`.

Claude Code plugin: `/swarm-delta-sync` wraps the local agent's
`/swarm/delta/*` HTTP endpoints so operators can install, run-once,
tail, and stop sync from any host that has the plugin installed.

## Consequences

### Positive
- Bandwidth scales with **change**, not state size — viable on the mesh.
- Idempotent and resumable; partial sync is safe to retry.

### Negative
- Two seeds with badly drifted clocks can produce ping-pong deltas
  until NTP convergence.

### Neutral
- Default interval (60 s) is a starting point; high-churn fleets tune
  down to 10 s, archival fleets push to 600 s.

## Alternatives considered

- **Full snapshot replication.** Rejected: bandwidth and flash wear.
- **External Raft cluster.** Rejected: RAM and operator complexity for
  a fleet that may go offline for hours at a time.

## Plugin invocation

- `/swarm-delta-sync` install, start, tail
- `/swarm-delta-sync --once`
- `/swarm-delta-sync --console "--peer 169.254.42.2 --interval 30"`
- `/swarm-delta-sync --stop`
- `/swarm-delta-sync --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- `cognitum-one/cogs:src/cogs/swarm-delta-sync/`
- ADR-001 (cogs-as-plugins architecture)
- `swarm-distributed-store`, `swarm-backup-restore`
