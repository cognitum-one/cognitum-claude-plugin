---
description: Inspect multi-Seed cluster health and peer state (ADR-040)
allowed-tools:
  - Bash
---

Multi-Seed cluster status from the perspective of THIS device.

Hit:
- `GET /api/v1/cluster/health` — overall cluster health
- `GET /api/v1/peers` — discovered peer devices
- `GET /api/v1/sync/stats` — sync delta counters

Format as one table per response. For peers, columns: peer UUID (short), last-seen, RTT (ms), sync-lag (epochs).

If `/api/v1/peers` returns an empty list, say so and stop — the device isn't part of a cluster, no further analysis needed.

If sync-lag on any peer exceeds 10 epochs, flag it 🟡 (drift warning) and suggest the user check that peer's network connectivity.

Use:
```
curl -sk -H "Authorization: Bearer $COGNITUM_SEED_TOKEN" $COGNITUM_SEED_BASE<endpoint>
```
