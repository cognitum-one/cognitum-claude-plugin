# ADR: Swarm Mesh Manager as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-mesh-manager`
**Category**: swarm

## Context

A user with three or four seeds in their home or shop has no easy way to see what is on the network and what each is running. The mesh layer (ADR-084) handles transport; `self-healing-mesh` handles failure response; what is missing is the *human-facing inventory* — who is up, what cog is on which seed, what version, what RTT.

This is the cog that makes the swarm legible to its operator.

## Decision

`swarm-mesh-manager` runs every 30 s, performs a service-discovery sweep over the local mesh (mDNS + the encrypted UDP control channel from ADR-084), and publishes a roster: `[{ seed_id, ip, mesh_rtt_ms, agent_version, running_cogs[], uptime_s }, ...]`. It does not push commands — installation/upgrade is delegated to the Claude Code plugin layer (`/swarm-mesh-manager --console "..."`), which calls back into the standard seed control endpoints.

## Consequences

### Positive
- Featured cog — the "where are my seeds" surface.
- 12 KB binary; discovery work is mostly waiting on packets.
- Read-only by design; cannot accidentally take down the swarm.

### Negative
- Does not authenticate beyond the mesh layer's existing key exchange; a compromised peer can lie about its cog list.

### Neutral
- Discovery is local-segment; cross-subnet swarms need a relay (out of scope).

## Alternatives considered

- **Cloud-side fleet console.** Rejected: requires connectivity; not aligned with the offline-first seed contract.
- **Bundle discovery into `self-healing-mesh`.** Rejected: separation of concerns — healer reacts, manager reports.

## Plugin invocation
- `/swarm-mesh-manager` install, start, tail
- `/swarm-mesh-manager --once`
- `/swarm-mesh-manager --console "--interval 30"`
- `/swarm-mesh-manager --stop`
- `/swarm-mesh-manager --logs`

## Resource budget
- Binary: ~420 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/swarm-mesh-manager/` | ADR-001 | ADR-084 (mesh) | self-healing-mesh
