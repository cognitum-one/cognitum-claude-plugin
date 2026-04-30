# ADR: Swarm Consensus as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `swarm-consensus`
**Category**: swarm

## Context

Some fleet actions are unsafe to apply unilaterally — enabling a cog
that touches every room, rotating mesh keys, accepting an OTA from a
new signer. Letting any single seed make those decisions is a
single-compromise-defeats-the-fleet failure mode. Operators need a
quorum primitive: propose a change, collect votes from peers, only
apply if a majority agrees.

This is an **infrastructure / control-plane cog** — it does not consume
sensor data. Its input is mesh peer state plus a proposal; its output
is a verdict transcript.

## Decision

`cog-swarm-consensus` runs an armhf binary on each participating
seed. Round timing is governed by `--interval` (default 60 s). The
proposing seed broadcasts `{proposal_id, action, expires_at}` to
`--peers`; every peer applies a local policy check, signs a vote, and
returns it. The proposer aggregates and, on majority `accept`, emits a
co-signed verdict that other cogs (e.g. `swarm-deploy`) gate on.
Implementation is single-decree Paxos-lite; the mesh's `raft` consensus
layer is reused for log ordering. JSON output is `{proposal_id,
action, votes: [{peer, vote, sig}], verdict, applied}`.

Claude Code plugin: `/swarm-consensus` wraps the local
`/swarm/consensus/*` endpoints — install, start, tail, propose, stop.

## Consequences

### Positive
- Critical changes require co-signed quorum, not a single private key.
- Proposals are time-boxed; a partitioned proposer cannot block
  forever.
- Verdicts are appendable to the witness chain for audit.

### Negative
- Adds latency to gated actions (one full round-trip across peers).

### Neutral
- Default quorum is simple majority; tune in agent config for
  stricter policies.

## Alternatives considered

- **Always-trust-local-operator.** Rejected: original problem.
- **Full Raft cluster.** Rejected: overkill for low-frequency
  proposals on RAM-constrained devices.

## Plugin invocation

- `/swarm-consensus` install, start, tail
- `/swarm-consensus --once`
- `/swarm-consensus --console "--peers 169.254.42.2,169.254.42.3 --propose enable-cog:presence"`
- `/swarm-consensus --stop`
- `/swarm-consensus --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- `cognitum-one/cogs:src/cogs/swarm-consensus/`
- ADR-001 (cogs-as-plugins architecture)
- `swarm-witness-federation`, `swarm-deploy`
