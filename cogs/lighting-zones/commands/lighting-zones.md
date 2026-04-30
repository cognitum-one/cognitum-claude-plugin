---
description: Install (if needed) and run the `lighting-zones` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /lighting-zones — Lighting Zones

Cognitum cog: **Lighting Zones**

Turns lights on and off as people move between rooms

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `lighting-zones` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"lighting-zones"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/lighting-zones/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/lighting-zones/logs?lines=5`) and report.

## Usage

```
/lighting-zones
/lighting-zones --once          # one-shot via /console with --once
/lighting-zones --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/lighting-zones --stop           # stop the cog on the seed
/lighting-zones --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"lighting-zones"}`
- `POST /api/v1/apps/lighting-zones/start`
- `POST /api/v1/apps/lighting-zones/stop`
- `POST /api/v1/apps/lighting-zones/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/lighting-zones/logs?lines=N`
- `GET  /api/v1/apps/lighting-zones/manifest`
- `GET  /api/v1/apps/lighting-zones/config`
- `PUT  /api/v1/apps/lighting-zones/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/lighting-zones/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/lighting-zones/`
