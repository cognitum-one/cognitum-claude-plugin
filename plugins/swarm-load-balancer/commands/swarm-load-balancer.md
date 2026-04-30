---
description: Install (if needed) and run the `swarm-load-balancer` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /swarm-load-balancer — Swarm Load Balancer

Cognitum cog: **Swarm Load Balancer**

Spread queries across seeds so no single one overloads

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `swarm-load-balancer` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"swarm-load-balancer"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/swarm-load-balancer/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/swarm-load-balancer/logs?lines=5`) and report.

## Usage

```
/swarm-load-balancer
/swarm-load-balancer --once          # one-shot via /console with --once
/swarm-load-balancer --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/swarm-load-balancer --stop           # stop the cog on the seed
/swarm-load-balancer --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"swarm-load-balancer"}`
- `POST /api/v1/apps/swarm-load-balancer/start`
- `POST /api/v1/apps/swarm-load-balancer/stop`
- `POST /api/v1/apps/swarm-load-balancer/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/swarm-load-balancer/logs?lines=N`
- `GET  /api/v1/apps/swarm-load-balancer/manifest`
- `GET  /api/v1/apps/swarm-load-balancer/config`
- `PUT  /api/v1/apps/swarm-load-balancer/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between load balancing cycles |
| `peers` | string | `` | Comma-separated IP addresses of peer seeds in the load pool |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/swarm-load-balancer/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/swarm-load-balancer/`
