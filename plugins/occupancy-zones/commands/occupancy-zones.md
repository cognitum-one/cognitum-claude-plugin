---
description: Install (if needed) and run the `occupancy-zones` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /occupancy-zones — Occupancy Zones

Cognitum cog: **Occupancy Zones**

Counts people in each room through walls

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `occupancy-zones` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"occupancy-zones"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/occupancy-zones/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/occupancy-zones/logs?lines=5`) and report.

## Usage

```
/occupancy-zones
/occupancy-zones --once          # one-shot via /console with --once
/occupancy-zones --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/occupancy-zones --stop           # stop the cog on the seed
/occupancy-zones --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"occupancy-zones"}`
- `POST /api/v1/apps/occupancy-zones/start`
- `POST /api/v1/apps/occupancy-zones/stop`
- `POST /api/v1/apps/occupancy-zones/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/occupancy-zones/logs?lines=N`
- `GET  /api/v1/apps/occupancy-zones/manifest`
- `GET  /api/v1/apps/occupancy-zones/config`
- `PUT  /api/v1/apps/occupancy-zones/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/occupancy-zones/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/occupancy-zones/`
