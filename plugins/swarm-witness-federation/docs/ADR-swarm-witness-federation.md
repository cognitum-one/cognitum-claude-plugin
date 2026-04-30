# ADR: Witness Chain Federation as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-witness-federation`
**Category**: swarm

## Category

Each seed maintains a local witness chain — append-only Merkle log of
notable events (cog firings, OTA installs, config changes). On its own
this proves *a* seed observed something; it does not prove other seeds
*also* observed it. Compromising a single device defeats the chain.

Operators in regulated environments (healthcare, food-service,
construction) need **cross-seed attestation**: a tamper-evident audit
trail that requires colluding peers to forge.

This is an **infrastructure / control-plane cog** — it does not consume
sensor data. Its input is the local witness head and peer witness
heads from the mesh; its output is a co-signed federated head.

## Decision

`cog-swarm-witness-federation` runs an armhf binary on every
participating seed. Each `--interval` seconds (default 30) it gossips
its current witness head to `--peers`, collects their heads, and builds
a federated Merkle root signed (Ed25519) by every peer that agrees.
The federated root is appended back into each peer's local chain, so
the chains *interlock*. JSON output is `{round_id, federated_root,
co_signers[], dissenters[]}`.

Claude Code plugin: `/swarm-witness-federation` wraps the local
`/swarm/witness/*` endpoints — install, start, tail, force a round,
stop.

## Consequences

### Positive
- Forgery requires colluding majority of peers, not just one device.
- Interlocking roots make rewriting one chain detectable from any
  other peer.
- Replays are useful evidence for audits and incident review.

### Negative
- Adds bandwidth proportional to peer count × interval.

### Neutral
- Federation does not prevent suppression — a seed can still refuse
  to attest. `swarm-consensus` is the right tool for that.

## Alternatives considered

- **Cloud-anchored chain.** Rejected: defeats air-gap deployments and
  introduces a censorable third party.
- **Single-seed witness only.** Rejected: original problem.

## Plugin invocation

- `/swarm-witness-federation` install, start, tail
- `/swarm-witness-federation --once`
- `/swarm-witness-federation --console "--peers 169.254.42.2,169.254.42.3 --interval 60"`
- `/swarm-witness-federation --stop`
- `/swarm-witness-federation --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- `cognitum-one/cogs:src/cogs/swarm-witness-federation/`
- ADR-001 (cogs-as-plugins architecture)
- `audit-logger`, `swarm-consensus`
