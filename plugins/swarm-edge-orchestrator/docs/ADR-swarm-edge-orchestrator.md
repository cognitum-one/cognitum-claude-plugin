# ADR: Edge Orchestrator as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-edge-orchestrator`
**Category**: swarm

## Context

A typical Cognitum deployment has one Pi Zero 2 W seed plus several
ESP32 sensor nodes streaming CSI / motion / mic features over UDP. As
the node count grows, individually configuring each ESP32 — firmware
version, sample rate, feature mask, alert thresholds — becomes
unmanageable. Operators want a control plane that treats the ESP32
fleet as one logical resource.

This is an **infrastructure / control-plane cog** — it does not itself
consume the sensor stream. Its input is each ESP32 node's heartbeat
and config response; its output is a fleet manifest plus per-node
config diffs.

## Decision

`cog-swarm-edge-orchestrator` runs an armhf binary on the seed. Every
`--interval` seconds (default 2) it polls all known ESP32 nodes,
listens on `--udp-port` (default 5005) for unsolicited heartbeats, and
reconciles each node's actual config against the desired manifest.
Drifted nodes get a config push; missing nodes are flagged; new nodes
auto-register if their key is in the trust list. JSON output is
`{nodes: [{id, fw, ok, last_seen_s, drift[]}], total_nodes,
unreachable[]}`.

Claude Code plugin: `/swarm-edge-orchestrator` wraps the local
`/swarm/edge/*` endpoints — install, start, tail, force a reconcile,
stop.

## Consequences

### Positive
- Single source of truth for ESP32 config across the fleet.
- Auto-detects firmware drift and surfaces it before sensor data
  goes wrong.
- 2 s default polling is fast enough to catch most node failures.

### Negative
- UDP is unauthenticated by default; rely on the trust list and the
  mesh's link encryption (ADR-084).

### Neutral
- The orchestrator does not push firmware itself; it triggers the
  ESP32's own OTA endpoint when the desired version differs.

## Alternatives considered

- **Per-node SSH config push.** Rejected: ESP32s have no SSH and the
  manual loop scales poorly past five nodes.
- **MQTT-driven config.** Rejected for v1: requires a broker; covered
  by `swarm-mqtt-bridge` for events instead of config.

## Plugin invocation

- `/swarm-edge-orchestrator` install, start, tail
- `/swarm-edge-orchestrator --once`
- `/swarm-edge-orchestrator --console "--interval 5 --udp-port 5005"`
- `/swarm-edge-orchestrator --stop`
- `/swarm-edge-orchestrator --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- `cognitum-one/cogs:src/cogs/swarm-edge-orchestrator/`
- ADR-001 (cogs-as-plugins architecture)
- `swarm-cluster-monitor`, `swarm-mqtt-bridge`
