---
description: Install (if needed) and run the `swarm-mesh-manager` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /swarm-mesh-manager — Swarm Mesh Manager

Cognitum cog: **Swarm Mesh Manager**

Find, connect, and monitor all seeds on your network

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `swarm-mesh-manager` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"swarm-mesh-manager"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/swarm-mesh-manager/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/swarm-mesh-manager/logs?lines=5`) and report.

## Usage

```
/swarm-mesh-manager
/swarm-mesh-manager --once          # one-shot via /console with --once
/swarm-mesh-manager --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/swarm-mesh-manager --stop           # stop the cog on the seed
/swarm-mesh-manager --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"swarm-mesh-manager"}`
- `POST /api/v1/apps/swarm-mesh-manager/start`
- `POST /api/v1/apps/swarm-mesh-manager/stop`
- `POST /api/v1/apps/swarm-mesh-manager/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/swarm-mesh-manager/logs?lines=N`
- `GET  /api/v1/apps/swarm-mesh-manager/manifest`
- `GET  /api/v1/apps/swarm-mesh-manager/config`
- `PUT  /api/v1/apps/swarm-mesh-manager/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `30` | Seconds between network discovery scans |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/swarm-mesh-manager/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/swarm-mesh-manager/`
