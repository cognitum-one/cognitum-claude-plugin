---
description: Install (if needed) and run the `swarm-delta-sync` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /swarm-delta-sync — Swarm Delta Sync

Cognitum cog: **Swarm Delta Sync**

Auto-sync data between seeds — only sends changes

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `swarm-delta-sync` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"swarm-delta-sync"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/swarm-delta-sync/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/swarm-delta-sync/logs?lines=5`) and report.

## Usage

```
/swarm-delta-sync
/swarm-delta-sync --once          # one-shot via /console with --once
/swarm-delta-sync --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/swarm-delta-sync --stop           # stop the cog on the seed
/swarm-delta-sync --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"swarm-delta-sync"}`
- `POST /api/v1/apps/swarm-delta-sync/start`
- `POST /api/v1/apps/swarm-delta-sync/stop`
- `POST /api/v1/apps/swarm-delta-sync/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/swarm-delta-sync/logs?lines=N`
- `GET  /api/v1/apps/swarm-delta-sync/manifest`
- `GET  /api/v1/apps/swarm-delta-sync/config`
- `PUT  /api/v1/apps/swarm-delta-sync/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `60` | Seconds between sync cycles |
| `peer` | string | `169.254.42.1` | IP address of the peer seed to sync with |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/swarm-delta-sync/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/swarm-delta-sync/`
