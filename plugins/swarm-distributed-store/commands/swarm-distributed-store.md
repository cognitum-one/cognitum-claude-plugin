---
description: Install (if needed) and run the `swarm-distributed-store` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /swarm-distributed-store — Distributed Vector Store

Cognitum cog: **Distributed Vector Store**

Spread data across seeds and search them all at once

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `swarm-distributed-store` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"swarm-distributed-store"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/swarm-distributed-store/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/swarm-distributed-store/logs?lines=5`) and report.

## Usage

```
/swarm-distributed-store
/swarm-distributed-store --once          # one-shot via /console with --once
/swarm-distributed-store --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/swarm-distributed-store --stop           # stop the cog on the seed
/swarm-distributed-store --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"swarm-distributed-store"}`
- `POST /api/v1/apps/swarm-distributed-store/start`
- `POST /api/v1/apps/swarm-distributed-store/stop`
- `POST /api/v1/apps/swarm-distributed-store/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/swarm-distributed-store/logs?lines=N`
- `GET  /api/v1/apps/swarm-distributed-store/manifest`
- `GET  /api/v1/apps/swarm-distributed-store/config`
- `PUT  /api/v1/apps/swarm-distributed-store/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between distribution cycles |
| `peers` | string | `` | Comma-separated IP addresses of peer seeds in the distributed store |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/swarm-distributed-store/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/swarm-distributed-store/`
