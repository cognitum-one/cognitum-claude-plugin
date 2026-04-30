---
description: Install (if needed) and run the `micro-hnsw` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /micro-hnsw — Micro HNSW

Cognitum cog: **Micro HNSW**

Fast on-device fingerprinting and classification

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `micro-hnsw` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"micro-hnsw"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/micro-hnsw/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/micro-hnsw/logs?lines=5`) and report.

## Usage

```
/micro-hnsw
/micro-hnsw --once          # one-shot via /console with --once
/micro-hnsw --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/micro-hnsw --stop           # stop the cog on the seed
/micro-hnsw --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"micro-hnsw"}`
- `POST /api/v1/apps/micro-hnsw/start`
- `POST /api/v1/apps/micro-hnsw/stop`
- `POST /api/v1/apps/micro-hnsw/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/micro-hnsw/logs?lines=N`
- `GET  /api/v1/apps/micro-hnsw/manifest`
- `GET  /api/v1/apps/micro-hnsw/config`
- `PUT  /api/v1/apps/micro-hnsw/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/micro-hnsw/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/micro-hnsw/`
