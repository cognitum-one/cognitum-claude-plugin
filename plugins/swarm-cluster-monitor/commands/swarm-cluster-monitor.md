---
description: Install (if needed) and run the `swarm-cluster-monitor` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /swarm-cluster-monitor — Cluster Health Monitor

Cognitum cog: **Cluster Health Monitor**

Live dashboard of every seed's health and status

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `swarm-cluster-monitor` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"swarm-cluster-monitor"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/swarm-cluster-monitor/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/swarm-cluster-monitor/logs?lines=5`) and report.

## Usage

```
/swarm-cluster-monitor
/swarm-cluster-monitor --once          # one-shot via /console with --once
/swarm-cluster-monitor --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/swarm-cluster-monitor --stop           # stop the cog on the seed
/swarm-cluster-monitor --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"swarm-cluster-monitor"}`
- `POST /api/v1/apps/swarm-cluster-monitor/start`
- `POST /api/v1/apps/swarm-cluster-monitor/stop`
- `POST /api/v1/apps/swarm-cluster-monitor/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/swarm-cluster-monitor/logs?lines=N`
- `GET  /api/v1/apps/swarm-cluster-monitor/manifest`
- `GET  /api/v1/apps/swarm-cluster-monitor/config`
- `PUT  /api/v1/apps/swarm-cluster-monitor/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `15` | Seconds between health polls |
| `peers` | string | `` | Comma-separated IP addresses of peer seeds to monitor |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/swarm-cluster-monitor/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/swarm-cluster-monitor/`
