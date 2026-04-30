# ADR: Audit Trail Logger as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `audit-logger`
**Category**: security

## Context

Compliance regimes (SOC 2, HIPAA, PCI-DSS, GDPR Art. 30) require a
tamper-evident record of every privileged action: cog install/stop,
config change, OTA event, peer-auth verdict, alarm fire. The seed
runs offline-capable in USB gadget mode and cannot rely on a cloud
log sink being reachable. A naïve append-only file is trivially
edited; centralized SIEMs (Splunk, Datadog) are not deployable to a
512 MB Pi Zero 2 W.

The relevant signal-processing approach is a **forward-secure hash
chain**: each log entry is `H(prev_hash || entry || timestamp)`. Any
truncation, reorder, or edit invalidates every hash from that point
forward. A periodic anchor digest is published to peers and (if
configured) forwarded to a cloud endpoint, so even root-on-seed cannot
silently rewrite history without breaking the externally-witnessed
anchor.

## Decision

Standalone armhf binary on seed. Tails the seed's event bus and
writes hash-chained JSONL records every 5 s default. Each chunk
ends with an anchor line containing the chunk's terminal hash, signed
with the seed's Ed25519 key. Optional `--cloud` flag streams anchors
(not full logs — privacy) to a configured HTTP endpoint. State
machine: `tailing → flushing → anchored`. Emits one JSON record per
event plus periodic `anchor` records. Packaged as Claude Code plugin:
slash command `/audit-logger` wraps seed's cog management endpoints.

## Consequences

### Positive
- Tamper-evident: any edit breaks the chain at verification time.
- Cloud-optional — operator chooses on-device only or anchor-forwarding.
- Tiny per-event overhead (one SHA-256 + small JSON write).

### Negative
- A determined attacker with root *can* delete the entire chain, but
  cannot edit it stealthily — the anchor gap is visible to peers.
- Not a substitute for a real SIEM at fleet scale; this is per-seed.

### Neutral
- Storage grows linearly. Operator must rotate or offload — a 5 s
  cadence at typical event rates fills the SD card in ~14 months.

## Alternatives considered

- **Plain syslog.** Rejected: trivially tamperable, no cryptographic
  integrity.
- **Blockchain anchor per event.** Rejected: bandwidth and cost out of
  proportion to a Pi Zero 2 W's role; chunked anchors are the right
  granularity.

## Plugin invocation

- `/audit-logger` — install if needed, start, tail logs
- `/audit-logger --once` — one-shot via `/console` with `--once`
- `/audit-logger --console "<args>"` — arbitrary args
- `/audit-logger --stop` — stop cog
- `/audit-logger --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/audit-logger/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `fleet-auth`, `prompt-shield`.
