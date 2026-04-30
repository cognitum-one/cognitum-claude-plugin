---
description: Install (if needed) and run the `plant-growth` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /plant-growth — Plant Growth

Cognitum cog: **Plant Growth**

Tracks plant growth rate and day/night cycles

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `plant-growth` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"plant-growth"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/plant-growth/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/plant-growth/logs?lines=5`) and report.

## Usage

```
/plant-growth
/plant-growth --once          # one-shot via /console with --once
/plant-growth --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/plant-growth --stop           # stop the cog on the seed
/plant-growth --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"plant-growth"}`
- `POST /api/v1/apps/plant-growth/start`
- `POST /api/v1/apps/plant-growth/stop`
- `POST /api/v1/apps/plant-growth/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/plant-growth/logs?lines=N`
- `GET  /api/v1/apps/plant-growth/manifest`
- `GET  /api/v1/apps/plant-growth/config`
- `PUT  /api/v1/apps/plant-growth/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `300` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/plant-growth/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/plant-growth/`
