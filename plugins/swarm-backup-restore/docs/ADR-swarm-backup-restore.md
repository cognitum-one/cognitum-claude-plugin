# ADR: Swarm Backup & Restore as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-backup-restore`
**Category**: swarm

## Context

Pi Zero 2 W SD cards fail. Single-seed deployments lose all learned
state — HNSW index, witness chain, cog parameters — when the card
goes. Operators want backups that live on **another seed**, not in the
cloud, so the appliance stays self-contained, and they want one-click
restore when a replacement card is imaged.

This is an **infrastructure / control-plane cog** — it does not consume
sensor data. Its input is the local AgentDB state directory; its
output is a backup manifest stored on the peer.

## Decision

`cog-swarm-backup-restore` runs an armhf binary on the seed being
protected. In `--mode backup` it streams an incremental, signed
snapshot to `--peer` every `--interval` seconds (default 120). In
`--mode restore` it pulls the most recent snapshot from `--peer` and
unpacks it into the agent's state directory before the agent starts.
JSON output is `{mode, peer, manifest_id, bytes, deltas, ok}`.

Claude Code plugin: `/swarm-backup-restore` wraps the local
`/swarm/backup/*` endpoints — install, start, tail (default mode is
backup), trigger a one-shot restore, stop.

## Consequences

### Positive
- Single peer covers many seeds (fan-in topology).
- Incremental — backup interval can be aggressive (every 2 min) without
  burning peer flash.
- Restore is one-click and verifies the Ed25519 signature.

### Negative
- Restore is whole-state, not partial — you cannot restore "just the
  witness chain".

### Neutral
- For more than one redundant peer, run multiple instances pointed at
  different `--peer` addresses.

## Alternatives considered

- **Cloud backup.** Rejected: defeats the air-gappable promise.
- **External NAS.** Rejected: extra infra; the mesh already has peers.

## Plugin invocation

- `/swarm-backup-restore` install, start, tail
- `/swarm-backup-restore --once`
- `/swarm-backup-restore --console "--peer 169.254.42.2 --mode backup --interval 300"`
- `/swarm-backup-restore --stop`
- `/swarm-backup-restore --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- `cognitum-one/cogs:src/cogs/swarm-backup-restore/`
- ADR-001 (cogs-as-plugins architecture)
- `swarm-delta-sync`, `swarm-distributed-store`
