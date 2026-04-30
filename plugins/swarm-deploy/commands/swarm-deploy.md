---
description: Install (if needed) and run the `swarm-deploy` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /swarm-deploy — Swarm Deploy

Cognitum cog: **Swarm Deploy**

Install or remove cogs on all seeds at once

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `swarm-deploy` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"swarm-deploy"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/swarm-deploy/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/swarm-deploy/logs?lines=5`) and report.

## Usage

```
/swarm-deploy
/swarm-deploy --once          # one-shot via /console with --once
/swarm-deploy --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/swarm-deploy --stop           # stop the cog on the seed
/swarm-deploy --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"swarm-deploy"}`
- `POST /api/v1/apps/swarm-deploy/start`
- `POST /api/v1/apps/swarm-deploy/stop`
- `POST /api/v1/apps/swarm-deploy/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/swarm-deploy/logs?lines=N`
- `GET  /api/v1/apps/swarm-deploy/manifest`
- `GET  /api/v1/apps/swarm-deploy/config`
- `PUT  /api/v1/apps/swarm-deploy/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `peers` | string | `` | Comma-separated IP addresses of peer seeds to deploy to |
| `install` | string | `` | Cog ID to install on all peers (e.g. cog-presence) |
| `uninstall` | string | `` | Cog ID to uninstall from all peers |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/swarm-deploy/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/swarm-deploy/`
