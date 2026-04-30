# ADR: ML Intrusion Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `intrusion-detect-ml`
**Category**: security

## Context

Signature-based network IDS (Snort rules, blocklists) catches known
attacks but misses novel ones — port-knocking variants, slow-scan
reconnaissance, low-and-slow exfiltration, and lateral movement that
mimics legitimate traffic. The seed sits at a privileged vantage on
the local network and can observe per-flow features (inter-arrival
times, packet-size histograms, TCP-flag distributions) without
deep-payload inspection. Cloud-based ML IDS exists but adds latency
and exfiltrates the very telemetry it should protect.

The signal-processing approach is a **quantized isolation-forest
classifier** (~14 KB model) over a 12-dim flow-feature vector
(packets/sec, bytes/sec, IAT mean/var, flag entropy, dst-port entropy,
SYN-to-ACK ratio, etc.). Inference is one tree-walk per flow per scan
tick — well within Pi Zero 2 W budget.

## Decision

Standalone armhf binary on seed. Every 3 s default, samples active
flows from `/proc/net/nf_conntrack` (or `ss -tn`), computes the
12-feature vector per flow, and runs the embedded isolation forest.
State machine: `learning → scoring → ALERT`. Outputs anomaly score
in `[0, 1]`; alerts above 0.7. Emits JSON with `flow_5tuple`,
`score`, `top_contributing_features`. Packaged as Claude Code plugin:
slash command `/intrusion-detect-ml` wraps seed's cog management
endpoints.

## Consequences

### Positive
- Catches novel attack patterns without rule updates.
- No payload inspection — TLS traffic is fine; only metadata is used.
- Quantized model fits in 14 KB; full cog binary is small enough for
  field OTA over the gadget link.

### Negative
- Forest must be retrained per-deployment to learn local-traffic norms;
  generic shipped model has higher false-positive rate.
- Score interpretability is limited — operator gets a feature attribution
  but not a "this is X attack" verdict.

### Neutral
- 3 s scan cadence trades detection latency against CPU. Default fits
  most deployments.

## Alternatives considered

- **Suricata + ET Open ruleset.** Rejected: too heavy for Pi Zero 2 W
  and signature-only.
- **Deep packet inspection ML.** Rejected: TLS opacity + RAM budget.
  Metadata is the right abstraction at the seed tier.

## Plugin invocation

- `/intrusion-detect-ml` — install if needed, start, tail logs
- `/intrusion-detect-ml --once` — one-shot via `/console` with `--once`
- `/intrusion-detect-ml --console "<args>"` — arbitrary args
- `/intrusion-detect-ml --stop` — stop cog
- `/intrusion-detect-ml --logs` — recent output

## Resource budget

- Binary: ~ 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one Pi Zero 2 W core.

## See also

- Source: `cognitum-one/cogs:src/cogs/intrusion-detect-ml/`
- Foundational: ADR-001 (cogs as plugins).
- Related cogs: `network-firewall`, `behavioral-profiler`.
