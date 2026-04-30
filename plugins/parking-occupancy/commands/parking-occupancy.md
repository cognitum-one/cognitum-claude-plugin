---
description: Install (if needed) and run the `parking-occupancy` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /parking-occupancy — Parking Occupancy

Cognitum cog: **Parking Occupancy**

Per-zone parking occupancy via ESP32 CSI subcarrier-amplitude shift. Tracks utilization and churn-per-hour. Requires ruview

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `parking-occupancy` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"parking-occupancy"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/parking-occupancy/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/parking-occupancy/logs?lines=5`) and report.

## Usage

```
/parking-occupancy
/parking-occupancy --once          # one-shot via /console with --once
/parking-occupancy --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/parking-occupancy --stop           # stop the cog on the seed
/parking-occupancy --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"parking-occupancy"}`
- `POST /api/v1/apps/parking-occupancy/start`
- `POST /api/v1/apps/parking-occupancy/stop`
- `POST /api/v1/apps/parking-occupancy/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/parking-occupancy/logs?lines=N`
- `GET  /api/v1/apps/parking-occupancy/manifest`
- `GET  /api/v1/apps/parking-occupancy/config`
- `PUT  /api/v1/apps/parking-occupancy/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Sampling interval |
| `zones` | integer | `4` | Subcarrier groups; each is one parking-zone proxy |
| `threshold` | float | `0.4` | Subcarrier-energy ratio above baseline-mean to count as occupied |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/parking-occupancy/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/parking-occupancy/`
