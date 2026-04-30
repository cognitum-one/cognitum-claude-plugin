# ADR: MQTT Bridge as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-mqtt-bridge`
**Category**: swarm

## Context

Many existing home-automation and industrial telemetry systems already
speak MQTT. Cognitum seeds publish events on the encrypted mesh, but
that's invisible to Home Assistant, Node-RED, ESP32 worker fleets, and
SCADA gateways. Forcing every external integrator to learn the mesh
RPC is a non-starter; operators want a one-cog bridge.

This is an **infrastructure / control-plane cog** — it does not consume
sensor data. Its input is the mesh event bus plus subscribed MQTT
topics; its output is fan-out across both transports.

## Decision

`cog-swarm-mqtt-bridge` runs an armhf binary on the seed. It connects
to the configured MQTT broker (defaults to local Mosquitto), subscribes
to a configurable topic prefix, and translates between the mesh event
format and MQTT messages. Every `--interval` seconds (default 5) it
also fans out to peers in `--peers` so multiple seeds share a unified
event view. JSON output is `{published, received, dropped,
broker_ok, peer_relays[]}`.

Claude Code plugin: `/swarm-mqtt-bridge` wraps the local
`/swarm/mqtt/*` endpoints — install, start, tail, stop.

## Consequences

### Positive
- Drop-in for Home Assistant, Node-RED, and other MQTT-native
  ecosystems; no custom integration code on their side.
- Bidirectional — external systems can also drive cog actions.

### Negative
- Adds an external broker dependency unless the local Mosquitto is
  used.

### Neutral
- QoS defaults to 1; tune in agent config for fire-and-forget
  telemetry vs. critical alerts.

## Alternatives considered

- **HTTP webhook fan-out.** Rejected: stateless, no replay, and
  operators were already running MQTT.
- **Custom WebSocket protocol.** Rejected: yet another wire format
  in a domain that already standardized on MQTT.

## Plugin invocation

- `/swarm-mqtt-bridge` install, start, tail
- `/swarm-mqtt-bridge --once`
- `/swarm-mqtt-bridge --console "--peers 169.254.42.2,169.254.42.3 --interval 10"`
- `/swarm-mqtt-bridge --stop`
- `/swarm-mqtt-bridge --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- `cognitum-one/cogs:src/cogs/swarm-mqtt-bridge/`
- ADR-001 (cogs-as-plugins architecture)
- `swarm-cluster-monitor`, `swarm-edge-orchestrator`
