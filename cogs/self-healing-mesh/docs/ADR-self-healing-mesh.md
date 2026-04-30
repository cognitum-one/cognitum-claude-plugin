# ADR: Self-Healing Mesh as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `self-healing-mesh`
**Category**: developer

## Context

Per ADR-084, seeds form a peer-to-peer encrypted UDP mesh. In real deployments — apartments, warehouses, farms — nodes go offline constantly: power blip, WiFi roam, kernel panic, dog unplugs the seed. Without active healing, dropped peers leave consumer cogs (e.g. multi-room presence, mesh fall-detect) silently underpowered until a human notices.

The seed already has neighbor discovery; what it lacks is a continuous reconciliation loop that detects departures, re-elects coverage, and rebalances cog assignments.

## Decision

`self-healing-mesh` runs every 10 s, snapshots the live peer set from the mesh module, diffs against the previous snapshot, and on departure: (a) marks the missing peer's covered zones as `degraded`, (b) elects the nearest live peer (by mesh-RTT) to take over each affected cog, (c) emits `mesh_rebalance` with the new assignments. On rejoin it reverses the takeover. All actions go through the existing mesh control channel — this cog is a controller, not a transport.

## Consequences

### Positive
- Fleet keeps working through individual node losses without operator intervention.
- 14 KB binary; piggybacks on existing mesh telemetry.
- Rejoin path prevents zone "ownership drift" after temporary outages.

### Negative
- A flapping node (rejoin-leave-rejoin) causes assignment churn; needs `mesh` feature flag and a hold-down timer to dampen.

### Neutral
- Healing decisions are local (each node runs its own copy and they converge); no central authority.

## Alternatives considered

- **Centralized mesh manager on the cloud control plane.** Rejected: defeats offline operation.
- **Static fallback assignments.** Rejected: doesn't adapt to deployment topology.

## Plugin invocation
- `/self-healing-mesh` install, start, tail
- `/self-healing-mesh --once`
- `/self-healing-mesh --console "--interval 10"`
- `/self-healing-mesh --stop`
- `/self-healing-mesh --logs`

## Resource budget
- Binary: ~430 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/self-healing-mesh/` | ADR-001 | ADR-084 (mesh) | swarm-mesh-manager
