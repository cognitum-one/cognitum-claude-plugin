---
description: Install (if needed) and run the `swarm-edge-orchestrator` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /swarm-edge-orchestrator — Edge Orchestrator

Cognitum cog: **Edge Orchestrator**

Manage all ESP32 sensor nodes from one place

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `swarm-edge-orchestrator` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"swarm-edge-orchestrator"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/swarm-edge-orchestrator/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/swarm-edge-orchestrator/logs?lines=5`) and report.

## Usage

```
/swarm-edge-orchestrator
/swarm-edge-orchestrator --once          # one-shot via /console with --once
/swarm-edge-orchestrator --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/swarm-edge-orchestrator --stop           # stop the cog on the seed
/swarm-edge-orchestrator --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"swarm-edge-orchestrator"}`
- `POST /api/v1/apps/swarm-edge-orchestrator/start`
- `POST /api/v1/apps/swarm-edge-orchestrator/stop`
- `POST /api/v1/apps/swarm-edge-orchestrator/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/swarm-edge-orchestrator/logs?lines=N`
- `GET  /api/v1/apps/swarm-edge-orchestrator/manifest`
- `GET  /api/v1/apps/swarm-edge-orchestrator/config`
- `PUT  /api/v1/apps/swarm-edge-orchestrator/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `2` | Seconds between node polls |
| `udp_port` | integer | `5005` | Port for ESP32 CSI data reception |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/swarm-edge-orchestrator/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/swarm-edge-orchestrator/`
