# ADR: Cluster Health Monitor as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-cluster-monitor`
**Category**: swarm

## Context

Once a household or worksite runs more than two seeds, "is everything
healthy?" stops being answerable by SSHing in. Operators need a
single live view of every peer's CPU, RAM, mesh RTT, OTA version, and
running cogs — without standing up Prometheus on a 512 MB device.

This is an **infrastructure / control-plane cog** — it does not read
sensor data. Its input is the mesh state plus each peer's
`/api/v1/health` and `/api/v1/cogs` endpoints; its output is a
flattened JSON dashboard frame.

## Decision

`cog-swarm-cluster-monitor` runs an armhf binary on a designated seed
(or all seeds — the work is idempotent). Every `--interval` seconds
(default 15) it polls each address in `--peers`, aggregates results,
and writes a JSON snapshot of the form `{peers: [{addr, ok, cpu_pct,
ram_mb, mesh_rtt_ms, version, cogs[]}], degraded[], unreachable[]}`.

Claude Code plugin: `/swarm-cluster-monitor` wraps the local
`/swarm/cluster/*` endpoints, surfaces a live tail in the operator's
terminal, and exits non-zero when any peer is unreachable so it can be
chained into CI/CD.

## Consequences

### Positive
- Zero-config dashboard for fleets up to ~20 seeds.
- Exit-code semantics make it scriptable for alerting.

### Negative
- Polling is O(peers) per interval; very large fleets should use
  `swarm-mqtt-bridge` push instead.

### Neutral
- Snapshots are not persisted by default; pair with `audit-logger` if
  you need history.

## Alternatives considered

- **Prometheus + node_exporter.** Rejected: RAM cost on Pi Zero 2 W
  and a separate scraper to operate.
- **Push-based heartbeat.** Rejected for v1 because pull is easier to
  debug; covered separately by `swarm-mqtt-bridge`.

## Plugin invocation

- `/swarm-cluster-monitor` install, start, tail
- `/swarm-cluster-monitor --once`
- `/swarm-cluster-monitor --console "--peers 169.254.42.2,169.254.42.3 --interval 10"`
- `/swarm-cluster-monitor --stop`
- `/swarm-cluster-monitor --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- `cognitum-one/cogs:src/cogs/swarm-cluster-monitor/`
- ADR-001 (cogs-as-plugins architecture)
- `swarm-mqtt-bridge`, `swarm-deploy`
