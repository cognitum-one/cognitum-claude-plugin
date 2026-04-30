# ADR: Network Firewall as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `network-firewall`
**Category**: security

## Context

Cogs are third-party-installable binaries running on the seed. Even
sandboxed, a misbehaving or malicious cog can attempt to phone home,
exfiltrate sensor data, or scan the local LAN. The seed has no
hardware MMU enforcement of network policy and the upstream Linux
distro's iptables surface is too broad for casual operators to manage.
Per-cog rules need to be declared, applied, and audited from the same
plugin abstraction the user already uses to install the cog.

The relevant signal-processing approach is a **periodic policy
reconciliation loop**: every scan interval the cog enumerates open
sockets owned by each running cog (via `/proc/net/tcp,udp` cross-ref
with PID->cog map), compares against the declared per-cog allowlist,
and emits a `block` event for anything outside. A threshold counter
(N violations within window M) escalates to active drop via nftables.

## Decision

Standalone armhf binary on seed. Every 10 s default, snapshots
per-cog socket state and compares against `network.allow` declared in
each cog's manifest. State machine: `monitoring → violation_seen →
ACTIVE_BLOCK`. First violation logs only; second within 60 s installs
an nftables rule scoped to the offending cog's cgroup. Emits JSON
with `cog_id`, `dst`, `verdict`, `rule_installed`. Packaged as Claude
Code plugin: slash command `/network-firewall` wraps seed's cog
management endpoints.

## Consequences

### Positive
- Per-cog policy, not global — a vetted cog can keep its allowed
  endpoints while a rogue cog is blocked.
- Two-strike rule prevents flapping on transient connection retries.
- Inspecting `/proc/net` is cheap; no kernel module required.

### Negative
- Polling at 10 s misses sub-second connect-and-exfiltrate bursts.
  Faster interval costs CPU.
- Cannot inspect TLS payload — only endpoint metadata.

### Neutral
- IPv6 and DNS-over-HTTPS bypass make endpoint allowlisting an
  approximate guarantee, not absolute.

## Alternatives considered

- **eBPF socket filter.** Rejected for v1: kernel surface too variable
  across seed images; Pi Zero 2 W kernel may lack BTF. Strong v2
  candidate.
- **Static iptables, no per-cog scoping.** Rejected: defeats the cog
  ecosystem's per-plugin trust model.

## Plugin invocation

- `/network-firewall` — install if needed, start, tail logs
- `/network-firewall --once` — one-shot via `/console` with `--once`
- `/network-firewall --console "<args>"` — arbitrary args
- `/network-firewall --stop` — stop cog
- `/network-firewall --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/network-firewall/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `intrusion-detect-ml`, `audit-logger`.
