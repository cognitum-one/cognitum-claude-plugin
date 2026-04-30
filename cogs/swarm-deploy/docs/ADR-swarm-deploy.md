# ADR: Swarm Deploy as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-deploy`
**Category**: swarm

## Context

Installing a cog one seed at a time scales poorly past three devices.
Operators routinely want to push a new cog (`presence`, `glass-break`,
etc.) to every seed in a household or worksite, or pull a misbehaving
cog from the entire fleet at once. Doing this by hand is slow and
error-prone; doing it by configuration management tools is overkill
for a 512 MB device.

This is an **infrastructure / control-plane cog** — it does not consume
sensor data. Its input is the mesh peer list plus the local cog
registry; its output is a per-peer deployment transcript.

## Decision

`cog-swarm-deploy` runs an armhf binary on the operator's seed and
fans out signed install/uninstall RPCs to every address in `--peers`
over the encrypted mesh tunnel. The OTA signature path (Ed25519,
ADR keys) is reused so peers refuse unsigned payloads. JSON output
is `{install: cog_id?, uninstall: cog_id?, results: [{peer, ok,
elapsed_ms, error?}]}`.

Claude Code plugin: `/swarm-deploy` wraps the local `/swarm/deploy/*`
endpoints. The plugin's `--console` form takes the same `--install` /
`--uninstall` flags as the binary, so a single command rolls a cog out
across the mesh.

## Consequences

### Positive
- One-shot fleet rollout / rollback in a single command.
- Reuses OTA signing — peers verify before applying.
- Per-peer results make partial failures actionable.

### Negative
- Atomic across-fleet rollout is **not** guaranteed; a partitioned
  peer simply gets the cog when it rejoins.

### Neutral
- For multi-step or dependent rollouts, prefer `swarm-consensus` to
  vote first.

## Alternatives considered

- **Ansible / SaltStack.** Rejected: external infra, doesn't speak
  the mesh, and requires SSH keys per seed.
- **Push via MQTT.** Rejected for binary payloads; MQTT bridge is
  used for events, not artifacts.

## Plugin invocation

- `/swarm-deploy` install, start, tail
- `/swarm-deploy --once`
- `/swarm-deploy --console "--peers 169.254.42.2,169.254.42.3 --install presence"`
- `/swarm-deploy --stop`
- `/swarm-deploy --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- `cognitum-one/cogs:src/cogs/swarm-deploy/`
- ADR-001 (cogs-as-plugins architecture)
- `swarm-consensus`, `swarm-cluster-monitor`
