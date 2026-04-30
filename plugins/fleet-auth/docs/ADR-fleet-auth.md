# ADR: Fleet Authentication as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `fleet-auth`
**Category**: security

## Context

A Cognitum fleet is N seeds in a mesh, each with its own Ed25519
identity keypair (ADR-084) and a rotating session credential for peer
links. Without ongoing verification a compromised or cloned seed can
silently rejoin the mesh, replay handshakes, or impersonate peers.
Manual cert pinning across a 20-seed deployment does not scale; cloud
identity providers (Auth0, Okta) do not work in the offline-capable USB
gadget mode the seed must support.

The relevant signal-processing approach is **threshold-based liveness
sampling**: each peer is challenged on a randomized interval, the
nonce-signed response timing forms a distribution, and persistent
deviation (z-score > 3 over 5 rounds) flags an impostor. Cert
expirations are tracked against the system clock, with a sliding
warn-window before expiry.

## Decision

Standalone armhf binary on seed. Every 30 s default, picks the
oldest-checked peer in the mesh, issues a challenge-response (Ed25519
sign-of-nonce), and updates a rolling reputation. Tracks cert
not-after timestamps and emits `cert_expiring`, `cert_expired`,
`peer_unreachable`, or `signature_invalid` events. State machine:
`idle → challenging → verified | suspect`. Emits JSON with
`peer_id`, `verdict`, `latency_ms`, `cert_days_remaining`. Packaged as
Claude Code plugin: slash command `/fleet-auth` wraps seed's cog
management endpoints.

## Consequences

### Positive
- Continuous liveness — a stolen-cert impostor is caught within minutes,
  not at next manual audit.
- Pure Ed25519, no PKI or external CA — works fully offline.
- Per-peer reputation persists across reboots in the seed's local store.

### Negative
- Challenge traffic adds modest mesh chatter (one round-trip per peer
  per check interval).
- Clock skew on a freshly-flashed seed can falsely flag "expired" certs
  until NTP converges.

### Neutral
- Default 30 s interval is a power-vs-detection trade-off; halve for
  high-security, double for low-traffic deployments.

## Alternatives considered

- **Cloud identity provider.** Rejected: offline gap, and seeds in USB
  gadget mode have no upstream reach.
- **One-time provisioning, no rechecks.** Rejected: a stolen seed would
  remain trusted indefinitely.

## Plugin invocation

- `/fleet-auth` — install if needed, start, tail logs
- `/fleet-auth --once` — one-shot via `/console` with `--once`
- `/fleet-auth --console "<args>"` — arbitrary args
- `/fleet-auth --stop` — stop cog
- `/fleet-auth --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/fleet-auth/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `audit-logger`, `prompt-shield`.
